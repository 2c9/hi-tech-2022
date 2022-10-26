## Astra (Strongswan) and REDOS (Libreswan) IKEv1

### REDOS

```bash
yum install libreswan
cat /etc/ipsec.conf
```

```
conn mytunnel
        auto=start
        authby=secret
        ikev2=no
        ike=aes128-sha256
        type=transport
        phase2=esp
        phase2alg=aes-sha2_256
        leftprotoport=gre
        rightprotoport=gre
        right=172.30.66.29
        left=%defaultroute
```

```bash
cat /etc/ipsec.secrets
```

```
172.30.66.62 %any : PSK 'Thisismysecret123x'
```

```bash
systemctl restart ipsec
```

### Astra Linux

```bash
apt install strongswan
cat /etc/ipsec.conf
```

```
conn mytunnel
        auto=start
        authby=secret
        keyexchange=ikev1
        ike=aes128-sha256-modp2048
        type=transport
        esp=aes-sha256-modp2048
        leftprotoport=gre
        rightprotoport=gre
        left=172.30.66.29
```

```bash
cat /etc/ipsec.secrets
```

```
%any %any : PSK 'Thisismysecret123x'
```