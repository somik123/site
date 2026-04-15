---
id: 13
title: Persistent Reverse Tunnel using AutoSSH
type: post
date: 2019-10-08 15:57:01
lastmod: 2023-04-01 23:36:39
categories:
- Bash
---


When the local server is behind a firewall with blocked ports or has dynamic IP or basically unreachable from the internet, it is possible to access the server using this method.




This requires a VPS or server with open ports and fixed IP address.







Login to remote server (ubuntu in our example) and append to file using vi or nano:





```bash
GatewayPorts yes
```



Then login to local server and install AutoSSH and SSH:





```bash
sudo apt update
sudo apt upgrade
sudo apt install autossh ssh
```



Then generate a SSH key pair for your server:





```bash
ssh-keygen
```




```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): */root/.ssh/nopwd*
Enter passphrase (empty for no passphrase): *leave empty*
Enter same passphrase again: *leave empty*
```



Note that for bottom part, -p 2222 means ssh is running on port 2222 for "remote" server.





```bash
ssh-copy-id -i .ssh/nopwd.pub -p 2222 root@remote.server.com
```



Create a connection file on local server and grant it permission to execute, and finally open it for editing:





```bash
sudo touch /root/start_conn.sh
sudo chmod +x /root/start_conn.sh
sudo nano /root/start_conn.sh
```




```bash
#!/bin/sh
autossh -M 10984 -N -f -i /root/.ssh/nopwd \
-o "PubkeyAuthentication=yes" -o "PasswordAuthentication=no" -o "ServerAliveInterval 60" \
-R 8080:192.168.1.50:80 \
-R 8443:192.168.1.50:443 \
-R 9001:192.168.1.50:10000 \
-R 9022:192.168.1.50:22 \
root@remote.server.com -p 2222 &
```





Exit and save by pressing **Ctrl + x**, followed by **Y** and **Enter**




Note that for above example, the line *-R 8080:192.168.1.50:80* means remote port 8080 will connect to local port 80. If no http client is running on remote server, it is possible to forward port 80 from remote to port 80 on local.




Otherwise, it is also possible to run a reverse proxy such as nginx proxy or apache proxy to forward specific domains to the correct port.




Finally initiate the connection on boot-up via cron:





```bash
crontabs -e
```




```bash
@reboot /root/start_conn.sh
```



And reboot the server:





```bash
sudo reboot
```


