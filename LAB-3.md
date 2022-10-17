### STEP 1 ) Install wireguard (CentOS)

```bash
yum install elrepo-release epel-release
yum install kmod-wireguard wireguard-tools
```

### STEP 2 ) Install wireguard (Ubuntu)

```bash
apt install wireguard
```

### STEP 3 ) Configure keys on the server and client

```bash
cd /etc/wireguard/
wg genkey | tee privatekey | wg pubkey > publickey
chmod 600 privatekey
vim wg0.conf
```

### STEP 4 ) Configure the server

```
### Sever config ###

[Interface]
Address = 192.168.5.1/24
ListenPort = 31194
PrivateKey = <ServerPriveKey>

[peer]
PublicKey = <ClientPubKey>
AllowedIPs = 192.168.5.2
```

### STEP 5 ) Configure the client

```
### Client config ###

[Interface]
PrivateKey = <ClientPrivKey>
Address = 192.168.5.2/24

[Peer]
PublicKey = <ServerPubKey>
AllowedIPs = 192.168.5.1, 10.10.30.0/24
Endpoint = 192.168.255.7:31194
```

### STEP 6 ) Restart the server and the client

```bash
systemctl enable --now  wg-quick@wg0
```
