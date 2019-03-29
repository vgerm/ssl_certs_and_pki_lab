# Lab01

## Build YOUR own LAB

### Open lab playground: [http://tools-cz1vg.cz.nonprod](http://tools-cz1vg.cz.nonprod)

### Add new Instance

### Clone git repo

```bash
git clone https://github.com/vgerm/ssl_certs_and_pki_lab.git
```

## Tips and Tricks

### Copy URL of YOUR LAB and don't loose it

If browser is accidentally closed, use the URL from above to restore your session!

### Ctrl+Ins (Copy) | Shift+Ins (Paste) | Use mouse

### Alt + Enter -> Full screen of terminal

### Connect via SSH

```bash
ssh -p 8022 ip10XXXXXXXXXX@direct.tools-cz1vg.cz.nonprod
```

### At the end of the day -> destroy/delete your lab and close your session

## Install OpenJDK

```bash
apk update
apk add openjdk8 nss
export JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
export PATH="$JAVA_HOME/bin:${PATH}"

java -version
javac -version
```
