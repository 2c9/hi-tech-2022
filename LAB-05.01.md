## Astra - openvswitch

### Fix repos

```bash
vim /etc/apt/sources.list
```
```
deb http://dl.astralinux.ru/astra/stable/1.7_x86-64/repository-main/ 1.7_x86-64 main contrib non-free
```

```bash
apt install -y apt-transport-https ca-certificates debian-archive-keyring dirmngr
vim /etc/apt/sources.list
```
```
deb https://mirror.yandex.ru/debian/ stretch main contrib non-free
```

### Install OVS

```bash
apt update
apt install openvswitch-switch
```

#### Unmanaged in NM

```bash
nmcli device set eth1 managed no
```

### Configure OVS

```bash
vim /etc/netwrok/interfaces
```

```
auto ovs0
allow-ovs ovs0
iface ovs0 inet manual
  ovs_type OVSBridge
  ovs_ports eth1 ovs0-10

allow-ovs0 eth1
iface eth1 inet manual
  ovs_type OVSPort
  ovs_bridge ovs0
  ovs_options vlan_mode=native-untagged tag=1 trunks=10,20,30

allow-ovs0 ovs0-10
iface ovs0-10 inet static
  ovs_bridge ovs0
  ovs_type OVSIntPort
  ovs_options tag=10
  address 192.168.40.1/24
```

### Trun on ovs

```bash
ifup ovs0
```