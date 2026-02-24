# SSL Certificate Creation with SAN

## Step 1: Create config `<SAN_config_file_name>.cnf`

```ini
[ req ]
default_bits       = 4096
prompt             = no
default_md         = sha256
req_extensions     = v3_req
distinguished_name = dn

[ dn ]
CN = *.gbkr.si

[ v3_req ]
keyUsage         = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName   = @alt_names

[ alt_names ]
DNS.1 = <some>.gbkr.si
DNS.2 = 
IP.1  = 
```

---

## Step 2: Generate private key and CSR

```bash
openssl req -new -newkey rsa:4096 -nodes \
  -keyout <key-name>.key \
  -out <csr_file_name>.csr \
  -config <SAN_config_file_name>.cnf \
  -extensions v3_req
```

> Submit `<csr_file_name>.csr` to the corporate CA for signing → receive `.p7b`

---

## Step 3: Convert `.p7b` → `.pem`

```bash
openssl pkcs7 -inform DER -print_certs \
  -in <file_name>.p7b \
  -out <server+root_certs>.pem
```

---

## Step 4: Append private key to `.pem` for HAProxy

```bash
cat <key-name>.key >> <server+root_certs>.pem
```

> Final `.pem` contains: server certificate + CA chain + private key