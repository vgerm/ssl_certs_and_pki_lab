# Lab06

## Troubleshooting/testing with openssl

```bash
openssl s_client -connect www.fd.com:443
openssl s_client -connect www.fd.com:443 -showcerts
openssl s_client -connect www.fd.com:443 -CAfile /etc/ssl/certs/ca-certificates.crt

openssl s_client -connect www.fd.com:443 -no_ssl2

openssl s_client -connect www.fd.com:443 -servername www.fd.com

openssl s_client -connect www.example.com:443 -tls1_2

openssl s_client -connect www.fd.com:443 -cipher RC4-SHA

echo | openssl s_client -connect www.fd.com:443 2>&1 | sed --quiet '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > www.fd.com.crt
openssl x509 -in fd.crt -noout -serial
openssl crl -in signing-ca.crl -inform DER -text -noout | grep FE760
```

## Troubleshooting/testing with curl

```bash
curl --verbose https://example.com
curl -k --verbose https://example.com
```

## Troubleshooting/testing with SSLPoke

```bash
cd src/
javac -version
javac SSLPoke.java

java SSLPoke www.fd.com 443

java \
    -Djavax.net.debug=ssl\
    SSLPoke www.fd.com 443

java \
    -Djavax.net.ssl.trustStoreType=jks \
    -Djavax.net.ssl.trustStore=./trustcacerts.jks \
    -Djavax.net.ssl.trustStorePassword=XXX \
    SSLPoke www.fd.com 443
```
