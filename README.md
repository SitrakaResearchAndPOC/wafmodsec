# wafmodsec
[link1](https://www.howtoforge.com/install-modsecurity-3-with-nginx-on-ubuntu-22-04/)
[link2](https://stackoverflow.com/questions/66205286/enable-systemctl-in-docker-container)
Terminal 1 : 
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
Terminal 2 : ctrl+shift+T
```
docker inspect wafmodsec3 | grep IPAddress
```
Terminal 1:
```
apt update 
```
```
apt install g++ flex bison curl apache2-dev doxygen libyajl-dev ssdeep liblua5.2-dev libgeoip-dev libtool dh-autoreconf libcurl4-gnutls-dev libxml2 libpcre++-dev libxml2-dev git liblmdb-dev libpkgconf3 lmdb-doc pkgconf zlib1g-dev libssl-dev -y
```
```
apt install wget
```
```
wget https://github.com/SpiderLabs/ModSecurity/releases/download/v3.0.8/modsecurity-v3.0.8.tar.gz
```
```
tar -xvzf modsecurity-v3.0.8.tar.gz
```
```
cd modsecurity-v3.0.8
```
```
./build.sh
```
```
./configure
```
```
make
```
```
make install
```
```
cd ~
```
```
git clone https://github.com/SpiderLabs/ModSecurity-nginx.git
```
```
wget https://nginx.org/download/nginx-1.20.2.tar.gz
```
```
tar xzf nginx-1.20.2.tar.gz
```
```
useradd -r -M -s /sbin/nologin -d /usr/local/nginx nginx
```
```
cd nginx-1.20.2
```
```
./configure --user=nginx --group=nginx --with-pcre-jit --with-debug --with-compat --with-http_ssl_module --with-http_realip_module --add-dynamic-module=/root/ModSecurity-nginx --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log
```
```
make
```
```
make modules
```
```
make install
```
```
ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/
```
```
nginx -V
```
```
cp ~/modsecurity-v3.0.8/modsecurity.conf-recommended /usr/local/nginx/conf/modsecurity.conf
```
```
cp ~/modsecurity-v3.0.8/unicode.mapping /usr/local/nginx/conf/
```
```
cp /usr/local/nginx/conf/nginx.conf{,.bak}
```
```
apt install nano
```
```
nano /usr/local/nginx/conf/nginx.conf
```
Remove default lines and add the following lines:
```

load_module modules/ngx_http_modsecurity_module.so;
user  nginx;
worker_processes  1;
pid        /run/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  nginx.example.com;
        modsecurity  on;
        modsecurity_rules_file  /usr/local/nginx/conf/modsecurity.conf;
        access_log  /var/log/nginx/access_example.log;
        error_log  /var/log/nginx/error_example.log;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
```
sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /usr/local/nginx/conf/modsecurity.conf
```
```
cd
```
```
git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git /usr/local/nginx/conf/owasp-crs
```
```
cp /usr/local/nginx/conf/owasp-crs/crs-setup.conf{.example,}
```
```
echo -e "Include owasp-crs/crs-setup.conf
Include owasp-crs/rules/*.conf" >> /usr/local/nginx/conf/modsecurity.conf
```
```
nginx -t
```
If everything is fine, you will get the following output:</br></br>
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok </br>
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful </br>
Creating service : </br>
nano /etc/systemd/system/nginx.service
```
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/local/nginx/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/local/nginx/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```
```
systemctl daemon-reload
```
```
systemctl start nginx
```
```
systemctl enable nginx
```
```
systemctl status nginx
```
You should see the following output: </br>
? nginx.service - A high performance web server and a reverse proxy server </br>
     Loaded: loaded (/etc/systemd/system/nginx.service; disabled; vendor preset: enabled) </br>
     Active: active (running) since Tue 2022-10-11 15:40:39 UTC; 6s ago </br>
       Docs: man:nginx(8) </br>
    Process: 68438 ExecStartPre=/usr/local/nginx/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS) </br>
    Process: 68439 ExecStart=/usr/local/nginx/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS) </br>
   Main PID: 68440 (nginx) </br>
      Tasks: 2 (limit: 4579) </br>
     Memory: 20.0M </br>
        CPU: 293ms </br>
     CGroup: /system.slice/nginx.service </br>
             ??68440 "nginx: master process /usr/local/nginx/sbin/nginx -g daemon on; master_process on;" </br>
             ??68441 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" > </br> </br> </br>

Oct 11 15:40:38 ubuntu2204 systemd[1]: Starting A high performance web server and a reverse proxy server... </br>
Oct 11 15:40:39 ubuntu2204 systemd[1]: Started A high performance web server and a reverse proxy server. </br> </br>

Verification : </br>
After installing and configuring Modsecurity with Nginx. It's time to test it. Run the following command to test the Modsecurity against command injection: </br>
```
curl localhost?doc=/bin/ls
```
If everything is fine, you will get the "403 Forbidden" massage.
```
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.20.2</center>
</body>
</html>
```
You can also check the Modesecurity log using the following command:
```
tail /var/log/modsec_audit.log
```
```
docker commit wafmodsec3 wafmodsec:v1
```
```
docker image save wafmodsec:v1 -o wafmodsec1.tar.gz
```
```
docker image save wafmodsec:v2 -o wafmodsec2.tar.gz
```
```
chmod 777 wafmodsec1.tar.gz
```
```
chmod 777 wafmodsec2.tar.gz
```
LOAD AN RUN
```
docker load -i  wafmodsec1.tar.gz
```
```
docker run -it  -p 80:80 --privileged --cap-add=ALL  wafmodsec:v1 --name wafmodsec3
```
login:root</br>
pass:root</br>
```
systemctl start nginx
```
```
systemctl enable nginx
```
```
systemctl status nginx
```
```
curl localhost?doc=/bin/ls
```
Treat other evasion technic

