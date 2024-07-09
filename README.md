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

باز کردن فایل زیر 


````
nano AddServ.py
````



=====================================================




````
#!/usr/bin/env python3

import subprocess

def read_haproxy_config(file_path):
    with open(file_path, 'r') as f:
        return f.readlines()

def write_haproxy_config(file_path, lines):
    with open(file_path, 'w') as f:
        f.writelines(lines)

def find_last_server_name(lines):
    server_names = [line.split()[1] for line in lines if line.strip().startswith('server')]
    if not server_names:
        return 'hostA'
    last_server_name = max(server_names)
    next_letter = chr(ord(last_server_name[-1]) + 1)
    return f"host{next_letter}"

def add_servers_to_ssh_section(file_path, servers):
    config_lines = read_haproxy_config(file_path)
    ssh_section_start = -1

    for i, line in enumerate(config_lines):
        if line.strip().startswith('listen ssh'):
            ssh_section_start = i
            break

    if ssh_section_start == -1:
        raise ValueError("بخش listen ssh در فایل پیدا نشد!")

    last_server_index = ssh_section_start

    for i, line in enumerate(config_lines[ssh_section_start:], start=ssh_section_start):
        if line.strip().startswith('server'):
            last_server_index = i

    insert_index = last_server_index + 1

    for server in servers:
        server_name = find_last_server_name(config_lines)
        new_server_line = f"        server {server_name} {server} check inter 10s fall 2 rise 1\n"
        config_lines.insert(insert_index, new_server_line)
        insert_index += 1

    write_haproxy_config(file_path, config_lines)

if __name__ == "__main__":
    haproxy_config_path = "/etc/haproxy/haproxy.cfg"  # مسیر فایل haproxy.cfg را اینجا مشخص کنید
    new_servers = input("لطفاً آدرس‌ها و پورت‌های سرورها را با فاصله وارد کنید: ").split()
    add_servers_to_ssh_section(haproxy_config_path, new_servers)

    # ریستارت سرویس haproxy
    subprocess.run(["systemctl", "restart", "haproxy"])
````



=====================================================

دستور ریستارت haproxy


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
python3 AddServ.py
````

=====================================================

بعد از هر تغییرات باید دستور زیر وارد شود

````
systemctl restart haproxy
````

=====================================================

باز کردن پورت و روشن کردن فایروال

````
ufw allow 22/tcp
ufw allow 22/udp
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


تماس تصویری


````
bash <(curl -Ls https://raw.githubusercontent.com/HamedAp/Ssh-User-management/master/ssh-calls.sh --ipv4)
````



=====================================================


مشکل نصب پکیج های ابونتو


````
nano /etc/apt/sources.list
````



=====================================================


تنظیم dns سرور


````
sudo nano /etc/resolv.conf
````

````
nameserver 8.8.8.8
nameserver 8.8.4.4
````




=====================================================


تنظیم ساعت سرور


````
apt install ntp
````

````
service ntp restart
````

````
sudo hwclock --hctosys 
````

````
https://askubuntu.com/questions/1096930/sudo-apt-update-error-release-file-is-not-yet-valid
````


````
dpkg-reconfigure tzdata
````



چک کردن مسیر دیتاسنتر 

````
https://bgp.tools/prefix/
````





ساخت mtproxy تلگرام
````
https://www.datisnetwork.com/run-mtproto.html
````
