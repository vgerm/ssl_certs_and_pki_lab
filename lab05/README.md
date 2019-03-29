# Lab05

## Pull Tomcat docker container

```bash
docker pull tomcat:8.5-alpine
```

## Create JKS keystore

- keystore.jks

```bash
cp keystore.jks tomcat-conf/
```

## Check tomcat configuration

```bash
# check if ssl options are enabled
cat tomcat-conf/server.xml
    <Connector
              protocol="org.apache.coyote.http11.Http11NioProtocol"
              port="8443" maxThreads="150"
              scheme="https" secure="true" SSLEnabled="true"
              keystoreFile="conf/keystore.jks" keystorePass="password"
              keyAlias="tomcat" clientAuth="false" sslProtocol="TLS" />
```

## Start tomcat docker container

```bash
cd tomcat-conf/
docker run -dit \
    --name tomcat \
    --rm \
    -p 8880:8080 \
    -p 8843:8443 \
    -v "$PWD/keystore.jks":/usr/local/tomcat/conf/keystore.jks \
    -v "$PWD/server.xml":/usr/local/tomcat/conf/server.xml \
    tomcat:8.5-alpine
```

## Test connectivity

```bash
curl http://ipXXXXXXX-8880.direct.tools-cz1vg.cz.nonprod
curl --verbose https://ipXXXXXXX-8843.direct.tools-cz1vg.cz.nonprod
curl -k --verbose https://ipXXXXXXX-8843.direct.tools-cz1vg.cz.nonprod
```
