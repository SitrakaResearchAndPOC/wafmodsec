# wafmodsec
[link1](https://www.howtoforge.com/install-modsecurity-3-with-nginx-on-ubuntu-22-04/)
[link2](https://stackoverflow.com/questions/66205286/enable-systemctl-in-docker-container)

```
apt-get install docker-buildx-plugin
```
```
nano Dockerfile
```
```
FROM ubuntu:22.04

RUN echo 'root:root' | chpasswd
RUN printf '#!/bin/sh\nexit 0' > /usr/sbin/policy-rc.d
RUN apt-get update
RUN apt-get install -y systemd systemd-sysv dbus dbus-user-session
RUN printf "systemctl start systemd-logind" >> /etc/profile

ENTRYPOINT ["/sbin/init"] 
```
```
docker build -t  wafmodsec -f Dockerfile .
```
```
docker run -it  -p 80:80 --privileged --cap-add=ALL   --name wafmodsec3 wafmodsec
```
```
docker inspect wafmodsec3 | grep IPAddress
```
```
apt update 
```


