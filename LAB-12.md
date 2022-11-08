## Astra gluster

```bash
wget -O - https://download.gluster.org/pub/gluster/glusterfs/5/rsa.pub | apt-key add -
deb https://download.gluster.org/pub/gluster/glusterfs/5/5.6/Debian/buster/amd64/apt/ buster main
```
DNS ну или

```bash
vim /et/hosts
172.16.97.75 astralinux-1
172.16.97.76 astralinux-2
172.16.97.77 astralinux-3
```

```bash
apt update
apt install -y glusterfs-server glusterfs-client glusterfs-common
```

```bash
mkfs.ext4 /dev/sdb
mkdir /glusterfs
vim /etc/fstab
```
```
/dev/sdb /glusterfs               ext4   defaults  0       1
```

```bash
mount -a
mkdir /glusterfs/gv0
```

```bash
gluster peer probe astralinux-2
gluster peer probe astralinux-3
```

```bash
gluster peer status
gluster pool list
```
```bash
gluster volume create gv0 replica 3 172.16.97.7{5,6,7}:/glusterfs/gv0
gluster volume start gv0
gluster volume info
```

```bash
vim /etc/fstab
```
```
localhost:/gv0 /srv glusterfs defaults,_netdev 0 0
```

```bash
systemctl daemon-reload
systemctl edit srv.mount
```

```
[Unit]
After=glusterd.service
Wants=glusterd.service
```


