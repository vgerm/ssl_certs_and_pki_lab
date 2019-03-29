# Lab02

## Openssl - Basic

```bash
openssl version
openssl version -a
openssl help
```

### Key and certificate management

- Key algorithm (RSA, DSA, ECDSA)
- Key size
- Passphrase

```bash
openssl genrsa -aes128 -out fd.key 2048
cat fd.key
openssl rsa -text -in fd.key
openssl rsa -in fd.key -pubout -out fd-public.key
cat fd-public.key
```

### Creating certificate signing requests (CSR)

```bash
openssl req -new -key fd.key -out fd.csr
```

#### Tip - unattended CSR generation

```bash
cat fd.cnf
[req]
prompt = no
distinguished_name = dn
req_extensions = ext
input_password = PASSPHRASE

[dn]
CN = www.fd.com
emailAddress = webmaster@fd.com
O = FD Ltd
L = Brno
C = CZ

[ext]
subjectAltName = DNS:www.fd.com,DNS:fd.com
```

```bash
openssl req -new -config fd.cnf -key fd.key -out fd.csr
```

### Examine CSR

```bash
openssl req -text -in fd.csr -noout
```

### Signing your own certificate (Self-Signed)

```bash
openssl x509 -req -days 365 -in fd.csr -signkey fd.key -out fd.crt
```

#### Tip - create self-signed certificate in one step

```bash
openssl req -new -x509 -days 365 -key fd.key -out fd.crt
```

or

```bash
openssl req -new -x509 -days 365 -key fd.key -out fd.crt \
-subj "/C=CZ/L=Brno/O=FD Ltd/CN=www.fd.com"
```

#### Tip - creating CSR from existing certificate

```bash
openssl x509 -x509toreq -in fd.crt -out fd.csr -signkey fd.key
```

### Creating certificate valid for multiple hostnames

subjectAltName = DNS:www.fd.com,DNS:fd.com

### Examining certificate

```bash
openssl x509 -text -in fd.crt -noout
```

## Openssl - Key and Certificate Conversion

### PEM and DER Conversion

**Binary (DER) certificate** - Contains an X.509 certificate in its raw form, using DER ASN.1 encoding.

**ASCII (PEM) certificate(s)** - Contains a base64-encoded DER certificate, with -----BEGIN CERTIFICATE----- used
as the header and -----END CERTIFICATE----- as the footer. Usually seen with only
one certificate per file, although some programs allow more than one certificate de-
pending on the context. For example, older Apache web server versions require the
server certificate to be alone in one file, with all intermediate certificates together in
another.

- PEM to DER

```bash
openssl x509 -inform PEM -in fd.crt -outform DER -out fd.der
```

- DER to PEM

```bash
openssl x509 -inform DER -in fd.der -outform PEM -out fd.pem
```

## Openssl - Private Certificate Authority (CA)

### Create Root CA

#### Create directories

```bash
mkdir -p ca/root-ca/private ca/root-ca/db crl certs
chmod 700 ca/root-ca/private
```

#### Create database

```bash
cp /dev/null ca/root-ca/db/root-ca.db
cp /dev/null ca/root-ca/db/root-ca.db.attr
echo 01 > ca/root-ca/db/root-ca.crt.srl
echo 01 > ca/root-ca/db/root-ca.crl.srl
```

#### Create CA request

```bash
openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca.csr \
    -keyout ca/root-ca/private/root-ca.key
```

#### Create CA certificate

```bash
openssl ca -selfsign \
    -config etc/root-ca.conf \
    -in ca/root-ca.csr \
    -out ca/root-ca.crt \
    -extensions root_ca_ext
```

### Create Signing CA

#### Create directories

```bash
mkdir -p ca/signing-ca/private ca/signing-ca/db crl certs
chmod 700 ca/signing-ca/private
```

#### Create database

```bash
cp /dev/null ca/signing-ca/db/signing-ca.db
cp /dev/null ca/signing-ca/db/signing-ca.db.attr
echo 01 > ca/signing-ca/db/signing-ca.crt.srl
echo 01 > ca/signing-ca/db/signing-ca.crl.srl
```

#### Create CA request

```bash
openssl req -new \
    -config etc/signing-ca.conf \
    -out ca/signing-ca.csr \
    -keyout ca/signing-ca/private/signing-ca.key
```

#### Create CA certificate

```bash
openssl ca \
    -config etc/root-ca.conf \
    -in ca/signing-ca.csr \
    -out ca/signing-ca.crt \
    -extensions signing_ca_ext
```

### Operate Signing CA

#### Create TLS server request

```bash
SAN=DNS:www.simple.org \
openssl req -new \
    -config etc/server.conf \
    -out certs/simple.org.csr \
    -keyout certs/simple.org.key
```

#### Create TLS server certificate

```bash
openssl ca \
    -config etc/signing-ca.conf \
    -in certs/simple.org.csr \
    -out certs/simple.org.crt \
    -extensions server_ext
```

#### Create TLS client request

```bash
openssl req -new \
    -config etc/client.conf \
    -out certs/fred.csr \
    -keyout certs/fred.key
```

#### Create TLS client certificate

```bash
openssl ca \
    -config etc/signing-ca.conf \
    -in certs/fred.csr \
    -out certs/fred.crt \
    -extensions client_ext
```

#### Revoke certificate (if needed)

```bash
openssl ca \
    -config etc/signing-ca.conf \
    -revoke ca/signing-ca/01.pem \
    -crl_reason superseded
```

#### Create CRL

```bash
openssl ca -gencrl \
    -config etc/signing-ca.conf \
    -out crl/signing-ca.crl
```

### Operate Root CA

#### Create CRL

```bash
openssl ca -gencrl \
    -config etc/root-ca.conf \
    -out crl/root-ca.crl
```

### Output Formats

#### Create DER certificate

```bash
openssl x509 \
    -in certs/fred.crt \
    -out certs/fred.cer \
    -outform der
```

#### Create PEM bundle (chain)

```bash
cat ca/signing-ca.crt ca/root-ca.crt > \
    ca/signing-ca-chain.pem

cat certs/fred.key certs/fred.crt > \
    certs/fred.pem
```

#### Create PKCS#12 bundle

```bash
openssl pkcs12 -export \
    -name "Fred Flintstone" \
    -inkey certs/fred.key \
    -in certs/fred.crt \
    -certfile ca/signing-ca-chain.pem \
    -out certs/fred.p12
```

## Updating default OS ca certificates

### Add Root CA to the default OS ca certificates store

```bash
cp ca/root-ca.crt /usr/local/share/ca-certificates
update-ca-certificates
```
