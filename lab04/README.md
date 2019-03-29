# Lab04

## Pull Apache docker container

```bash
docker pull httpd:2.4-alpine
```

## Create server certificate and certificate chain

- server.crt
- server.key
- server-ca.crt

```bash
cp server.crt server.key server-ca.crt apache-conf/
```

## Check httpd configuration

```bash
# check if ssl options are enabled
cat apache-conf/httpd.conf
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
LoadModule ssl_module modules/mod_ssl.so
Include conf/extra/httpd-ssl.conf

# check ssl configuration
cat apache-conf/httpd-ssl.conf
SSLCertificateFile "/usr/local/apache2/conf/server.crt"
SSLCertificateKeyFile "/usr/local/apache2/conf/server.key"
SSLCertificateChainFile "/usr/local/apache2/conf/server-ca.crt"
```

## Start apache docker container

```bash
cd apache-conf/
docker run \
    -dit \
    --name httpd \
    --rm \
    -p 8080:80 \
    -p 8043:443 \
    -v "$PWD/httpd.conf":/usr/local/apache2/conf/httpd.conf \
    -v "$PWD/httpd-ssl.conf":/usr/local/apache2/conf/extra/httpd-ssl.conf \
    -v "$PWD/server.crt":/usr/local/apache2/conf/server.crt \
    -v "$PWD/server.key":/usr/local/apache2/conf/server.key \
    -v "$PWD/server-ca.crt":/usr/local/apache2/conf/server-ca.crt \
    httpd:2.4-alpine
```

## Test connectivity

```bash
curl http://ipXXXXXXX-8080.direct.tools-cz1vg.cz.nonprod
curl --verbose https://ipXXXXXXX-8043.direct.tools-cz1vg.cz.nonprod
curl -k --verbose https://ipXXXXXXX-8043.direct.tools-cz1vg.cz.nonprod
```
