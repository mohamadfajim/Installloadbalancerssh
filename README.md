# Installloadbalancerssh
#نصب لود بالانسر ssh

به نام خدا

=====================================================

اول آپدیت میکنیم

````
apt update
````

=====================================================

دوباره دستور زیر برای نصب اپدیت

````
apt upgrade -y
````

=====================================================

نصب haproxy 

````
apt install haproxy -y
````

=====================================================

باز کردن فایل کانفیگ

````
nano /etc/haproxy/haproxy.cfg
````

=====================================================

تمام محتویات فایل کانفیگ پاک کرده و موارد زیر را واردش میکنیم

````
global
    log /dev/log local0
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30m
    maxconn 2500
    user haproxy
    group haproxy
    daemon

defaults
  log  global
  mode  tcp
  timeout connect 10s
  timeout client 36h
  option  dontlognull
  errorfile 400 /etc/haproxy/errors/400.http
  errorfile 403 /etc/haproxy/errors/403.http
  errorfile 408 /etc/haproxy/errors/408.http
  errorfile 500 /etc/haproxy/errors/500.http
  errorfile 502 /etc/haproxy/errors/502.http
  errorfile 503 /etc/haproxy/errors/503.http
  errorfile 504 /etc/haproxy/errors/504.http

listen ssh 
  bind *:28
  balance leastconn
  mode tcp
  option tcp-check  
  tcp-check expect rstring SSH-2.0-OpenSSH.*
    
  server hostA  138.201.159.178:28 check inter 10s fall 2 rise 1


  # lots more hosts here (we currently have 42 hosts in here)

listen stats
  mode http
  maxconn 15
  bind *:80
  stats enable
  stats show-node
  stats uri /stats
  stats refresh 10s
  stats auth adminname:adminpassword
  stats hide-version
````


در این فایل کانفیگ میتوانید فایل پورت اتصال و سرور هارا ویرایش کنید 


=====================================================

بعد از هر تغییرات باید دستور زیر وارد شود

````
systemctl restart haproxy
````

=====================================================

نمایش فایل دایرکتوری

````
ls
````

=====================================================

بعد آدرس آی‌پی و پورت سرور جدید به این صورت وارد کن
IP:port
مثلا
73.285.38.24:22



````
python3 serveradd.py
````

=====================================================

بعد از هر تغییرات باید دستور زیر وارد شود

````
systemctl restart haproxy
````

=====================================================

باز کردن پورت و روشن کردن فایروال

````
ufw allow 80/tcp
ufw allow 80/udp
ufw allow 443/tcp
ufw allow 443/udp
ufw allow 7301/tcp
ufw allow 7301/udp
ufw allow 28/tcp
ufw allow 28/udp
ufw enable
````

=====================================================
