### Install lxc

```bash
dnf install -y epel-release
dnf install -y lxc lxc-templates
systemctl enable --now lxc.serivce
```

### Create a container

http://images.linuxcontainers.org/

```bash
lxc-create --template download -n container-1 -- --dist centos --release 8-Stream --arch amd64 --keyserver hkp://keyserver.ubuntu.com
```
Change root's passowrd

```bash
chroot /var/lib/lxc/container-1/rootfs/
passwd
```

```bash
vim /var/lib/lxc/container-1/config
```

### Connect to the bridge

```bash
nmcli connection add \
	type bridge \
	ifname lxcbr0 \
	con-name lxcbr0 \
	ipv4.method \
	manual ipv4.addresses "10.10.10.1/24"
nmcli connection up lxcbr0
```

### Start the container
```bash
lxc-start -n container-1
lxc-console -n container-1 # Press 'Ctrl-a + q' ot escape
lxc-stop -n container-1
```

### Start a simple DNS/DHCP server
```bash
vim /etc/dnsmasq.d/simple
```
```
listen-address=10.10.10.1
server=8.8.8.8
dhcp-authoritative
dhcp-range=lxcbr0,10.10.10.50,10.10.10.150,1h
```
```bash
systemctl restart dnsmasq.serivce
```

### Configure NAT

```bash
nft add table nat
nft 'add chain nat postrouting { type nat hook postrouting priority 100 ; }'
nft 'add chain nat prerouting { type nat hook prerouting priority -100 ; }'
nft 'add rule nat postrouting oifname ens192 masquerade'
nft 'add rule nat prerouting iifname ens192 tcp dport 80 dnat to 10.10.10.84:8000'
```
Save chnages
```bash
nft list ruleset >> /etc/sysconfig/nftables.conf
```
Enable persistence
```bash
systemctl enable nftables
```

### Make IPv4 address static
```bash
vim /etc/dnsmasq.d/simple
```
```
dhcp-host=00:16:3e:09:c7:18,container1,10.10.10.84
```
```bash
systemctl reload dnsmasq
```