### Create CA conf files and certificates

```bash
mkdir /root/ca
cd /root/ca
mkdir certs newcerts private crl
touch index.txt
echo -n '00' > serial
vim ca.cnf
```

```ini
[ ca ]
default_ca = CA_default

[ CA_default ]
dir             = /root/ca
certs           = $dir/certs
crl_dir         = $dir/crl
database        = $dir/index.txt
new_certs_dir   = $dir/newcerts
certificate     = $dir/cacert.pem
serial          = $dir/serial
crlnumber       = $dir/crlnumber
crl             = $dir/crl.pem
private_key     = $dir/private/cakey.pem

copy_extensions = copy

default_days    = 365                   # how long to certify for
default_crl_days= 30                    # how long before next CRL
default_md      = sha256

policy          = policy_match

[ policy_match ]
countryName             = match
stateOrProvinceName     = optional
localityName            = optional
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[req]
distinguished_name = req_distinguished_name
x509_extensions = ca_ext

[req_distinguished_name]
countryName                     = Country Name (2 letter code)
countryName_default             = RU
localityName                    = Locality Name (eg, city)
localityName_default            = Moscow
0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = KP11
commonName                      = Common Name (e.g. server FQDN or YOUR name)
commonName_max                  = 64

[ ca_ext ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical,CA:true
```

```bash
openssl req -nodes \
            -new \
            -out cacert.csr \
            -keyout private/cakey.pem \
            -extensions ca_ext \
            -config ca.cnf

openssl ca -selfsign \
           -in cacert.csr \
           -out cacert.pem \
           -extensions ca_ext \
           -config ca.cnf
```

### Create server's certificate

```ini
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C  = RU
L  = Moscow
O  = KP11
CN = ht2022.wsr
[v3_req]
keyUsage = nonRepudiation, keyEncipherment, digitalSignature
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = ht2022.wsr
DNS.2 = dev.ht2022.wsr
DNS.3 = jenkins.ht2022.wsr
```

```bash
openssl req -nodes -newkey rsa:2048 -out ht.req -keyout ht.key -config ht.cnf
openssl ca -in ht.req -out ht.crt -config ca.cnf
```

### Make nginx use ssl

```nginx
server {
   listen 443 ssl;

   ...

   ssl_certificate     /etc/nginx/ht.crt;
   ssl_certificate_key /etc/nginx/ht.key;
}

server {
    listen 80 default_server;
    server_name _;
    return 301 https://$host$request_uri;
}
```

### Revoke a certificate

1. Grep /etc/ssl/index.txt to obtain the serial number of the key to be revoked
2. openssl ca -revoke /root/ca/newcerts/<SERIAL_NUMBER>.pem -config ca.cnf