### STEP 1) create namespaces
```bash
ip netns add ns1
ip netns add ns2
```

### STEP 2) create veth links and add peers to ns
```bash
ip link add veth1 type veth peer name eth0
ip link set eth0 netns ns1

ip link add veth2 type veth peer name eth0
ip link set eth0 netns ns2
```

### STEP 3) setup veth link
```bash
ip link set veth1 up
ip link set veth2 up
ip netns exec ns1 ip link set eth0 up
ip netns exec ns2 ip link set eth0 up
```

### STEP 4) assign ip address
```bash
ip netns exec ns1 ip addr add 10.10.10.11/24 dev eth0
ip netns exec ns2 ip addr add 10.10.10.12/24 dev eth0
```

### STEP 5) setup bridge
```bash
ip link add br0 type bridge
ip link set br0 up
```

### STEP 6) assign veth pairs to bridge
```bash
ip link set veth1 master br0
ip link set veth2 master br0
```

### STEP 7) setup bridge ip
```bash
ip addr add 10.10.10.1/24 dev br0
```

### STEP 8) add default routes for ns
```bash
ip netns exec ns1 ip route add default via 10.10.10.1
ip netns exec ns2 ip route add default via 10.10.10.1
```
### STEP 9) Let them out
```bash
sysctl -w net.ipv4.ip_forward=1
```

### STEP 10) Internet inside the namespaces
```bash
nft add table nat
nft 'add chain nat postrouting { type nat hook postrouting priority 100 ; }'
nft add rule nat postrouting oifname eth0 masquerade
```

### STEP 11) Run web servers
```bash
mkdir /srv/web{1,2}
echo '<h1>NS1</h1>' > /srv/web1/index.html
echo '<h1>NS2</h1>' > /srv/web2/index.html

ip netns exec ns1 bash -c 'cd /srv/web1; python3 -m http.server 8000' &
ip netns exec ns2 bash -c 'cd /srv/web2; python3 -m http.server 8000' &

curl http://10.10.10.11:8000
curl http://10.10.10.12:8000
```