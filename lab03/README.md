# Lab03

## Keytool basic

```bash
keytool -help
keytool -printcert -help
keytool -list -help
```

## Creating JKS keystore with a key and a self-signed certificate

```bash
keytool -genkey \
    -keyalg RSA \
    -alias tomcat \
    -keystore keystore.jks \
    -storepass password \
    -validity 365 \
    -keysize 2048
```

- with subjectAltName (-ext not available in Java 6 and earlier versions)

```bash
keytool -genkeypair \
    -keyalg RSA \
    -alias tomcat \
    -keystore keystore.jks \
    -storepass password \
    -validity 365 \
    -keysize 2048 \
    -ext SAN="DNS:www.fd.com,DNS:fd.com"
```

### Examine JKS keystore

```bash
keytool \
    -keystore keystore.jks \
    -list \
    -v
```

## Creating a CSR

```bash
keytool -certreq \
    -keystore keystore.jks \
    -alias tomcat \
    -file fd.csr
```

## Importing certificates

### Import the Root CA certificate

```bash
keytool -import \
    -keystore keystore.jks \
    -trustcacerts \
    -alias root-ca \
    -file ca/root-ca.crt
```

### Import the Signing CA certificate

```bash
keytool -import \
    -keystore keystore.jks \
    -trustcacerts \
    -alias signing-ca \
    -file ca/signing-ca.crt
```

### Import the server certificate

```bash
keytool -import \
    -keystore keystore.jks \
    -alias tomcat \
    -file fd.crt
```

## Updating default OS ca certificates

### Add Root CA to the default java cacerts

```bash
keytool -list -keystore /usr/lib/jvm/java-1.8-openjdk/jre/lib/security/cacerts
keytool -import \
    -keystore /usr/lib/jvm/java-1.8-openjdk/jre/lib/security/cacerts \
    -trustcacerts \
    -alias root-ca \
    -file ca/root-ca.crt
keytool -list -keystore /usr/lib/jvm/java-1.8-openjdk/jre/lib/security/cacerts
```
