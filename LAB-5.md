## Centos-0

### Configure NAT

```bash
dnf install -y nftables

nft add table nat
nft 'add chain nat postrouting { type nat hook postrouting priority 100 ; }'
nft 'add rule nat postrouting oifname eth0 masquerade'

nft list ruleset >> /etc/sysconfig/nftables.conf
systemctl enable nftables
```

### Enable forwarding

```bash
sysctl -w net.ipv4.ip_forward=1 > /etc/sysctl.d/route.conf
```

### Create VLAN profiles for internal networks

- VLAN 10 - MGMT
- VLAN 20 - DATA
- VLAN 30 - OFFICE
- VLAN 50 - Native (PVID, UNTAGGED)

```bash
nmcli connection add con-name eth1 \
                     ifname eth1 \
                     type ethernet \
                     ipv4.method manual \
                     ipv4.addresses 10.10.50.1/24

nmcli connection add con-name eth1.10 \
                     ifname eth1.10 \
                     type vlan \
                     dev eth1 \
                     id 10 \
                     ipv4.method manual \
                     ipv4.addresses 10.10.10.1/24
                     ipv6.method disabled

nmcli connection add con-name eth1.30 \
                     ifname eth1.30 \
                     type vlan \
                     dev eth1 \
                     id 30 \
                     ipv4.method manual \
                     ipv4.addresses 10.10.30.1/24
                     ipv6.method disabled
```

### Configure DHCP for internal netowrks

```bash
dnf install -y dnsmasq

vim /etc/dnsmasq.d/simple
```

```
listen-address=10.10.50.1,10.10.10.1,10.10.30.1

# DNS
server=8.8.8.8
# expand-hosts
# domain=example.lan
# address=/example.lan/10.10.10.50

# DHCP
dhcp-authoritative
dhcp-range=eth1,10.10.50.50,10.10.50.150,1h
dhcp-option=interface:eth1,3,10.10.50.1

dhcp-range=eth1.10,10.10.10.50,10.10.10.150,1h
dhcp-option=interface:eth1.10,3,10.10.10.1

dhcp-range=eth1.30,10.10.30.50,10.10.30.150,1h
dhcp-option=interface:eth1.30,3,10.10.30.1

dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases
```

Start and enable dnsmasq

```bash
systemctl enable --now dnsmasq.service
```

## iosvl2-0

```
!
hostname switch
line console 0
 exec-timeout 0 0
 logging synchronous 
!
vlan 10
 name MGMT
!
vlan 20
 name DATA
!
vlan 30
 name OFFICE
!
vlan 50
 name NATIVE
!
spanning-tree mode mst
!
interface gigabitEthernet 0/0
 switchport trunk encapsulation dot1q 
 switchport mode trunk
 switchport trunk native vlan 50
!
interface range gigabitEthernet 0/2-3
 channel-group 1 mode active
!
interface range gigabitEthernet 1/0-1
 channel-group 2 mode passive
!
interface po1
 switchport trunk encapsulation dot1q 
 switchport mode trunk
 switchport trunk native vlan 50
!
interface po2
 switchport trunk encapsulation dot1q 
 switchport mode trunk
 switchport trunk native vlan 50
!
```

## debian-0

### Install OVS

```bash
apt-get install -y openvswitch-switch
reboot
```

## Create a bridge

```bash
ifdown ens2
ifdown ens3

ovs-vsctl add-br br0
ovs-vsctl set Bridge br0 rstp_enable=false  # Does not work over lacp =(((
ovs-vsctl set Bridge br0 stp_enable=false  # Does not work over lacp =(((
ovs-vsctl show
ovs-appctl rstp/show
```

### Configure Link Aggregation and LACP ( IEEE 802.1AX-2008 )

```bash
ovs-vsctl add-bond br0 bond0 ens2 ens3 lacp=active
ovs-vsctl show
ovs-appctl lacp/show bond0
```

### Configure VLANs (802.1q)

```bash
# Trunks
ovs-vsctl set port bond0 vlan_mode=native-untagged tag=50 trunks=10,20,30
# Clients
ovs-vsctl add-port br0 ens9 tag=10
# SVI
ovs-vsctl add-port br0 SVI50 tag=50 -- set interface SVI50 type=internal
# Show the bridge's configuration
ovs-vsctl show
```

### Change INTERFACES

```bash
vim /etc/network/interfaces.d/50-cloud-init 
```

```
auto lo
iface lo inet loopback

auto SVI50
allow-hotplug SVI50
iface SVI50 inet dhcp
```

```bash
ifup SVI50
```

## centos-1

### Configure Link Aggregation and LACP ( IEEE 802.1AX-2008 )
```bash
nmcli connection add type team \
	 con-name team0 \
	 ifname team0 \
	 config '{"runner": {"name": "lacp", "active": true, "fast_rate": true }}' \
	 ipv4.method disabled \
     ipv6.method disabled

nmcli connection add type ethernet slave-type team con-name team0-port1 ifname eth0 master team0
nmcli connection add type ethernet slave-type team con-name team0-port2 ifname eth1 master team0
```
## Create a bridge

```bash
nmcli connection add type bridge \
      ifname br0 con-name br0 \
      bridge.vlan-filtering yes \
      bridge.vlan-default-pvid 50 \
      bridge.vlans "10,20,30,50 pvid untagged"
```

## Connect interfaces

```bash
nmcli connection modify team0 \
	 slave-type bridge \
	 master br0 \
	 bridge-port.vlans "10,20,30,50 pvid untagged"

nmcli connection add \
	type ethernet \
	con-name eth9 \
	slave-type bridge \
	master br0 \
	bridge-port.vlans "30 pvid untagged"
```