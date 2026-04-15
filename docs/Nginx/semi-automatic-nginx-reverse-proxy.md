---
id: 16
title: Semi-Automatic Nginx Reverse Proxy
type: post
date: 2019-10-08 16:06:33
lastmod: 2023-04-01 23:35:30
categories:
- Bash
---


When you have servers with IP addresses or running on different port then usual 80 and 443, you can hide it using a Nginx reverse proxy. From outside, it'll look just like a regular website with a normal domain or subdomain name, while Nginx will redirect the query to your actual server running behind it. It is usefull for using 1 static IP for multiple servers or for using a distributed load network.




This requires a VPS or server with open ports and fixed IP address.







Start by installing nginx, nano and certbot on the server (ubuntu in our example). The certbot will be used to get SSL certificates from LetsEncrypt for the domains you access through proxy.





```bash
sudo apt update
sudo apt upgrade
sudo apt install nginx nano python-certbot-nginx
```



Add the proxy configuration to Nginx:





```bash
sudo nano /etc/nginx/proxy.conf
```





Add the following content:







```bash
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_read_timeout 900s;
```





Exit and save by pressing **Ctrl + x**, followed by **Y** and **Enter**




Create the semi-automation script on your server and give it permission to execute:





```bash
sudo touch /root/add_server.sh
sudo chmod +x /root/add_server.sh
sudo nano /root/add_server.sh
```



Add the following content:





```bash
#!/bin/bash

read -p 'Enter the domain name (without www): ' DOMAIN
read -p 'Enter the full forwarded URL: ' FORDED_URL
read -p 'Enter the forwarded Hostname: ' FORDED_HOST

echo "Adding domain $DOMAIN to Nginx configuration"

cat <<EOF > /etc/nginx/sites-available/$DOMAIN.conf
server {
	server_name $DOMAIN;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location /.well-known {
        root /var/www/ssl/$DOMAIN/;
    }
    location / {
        include proxy.conf;
        proxy_pass $FORDED_URL;
		proxy_set_header Host $FORDED_HOST;

    }

    listen 80;
}

EOF

echo "Enabling domain"
ln -s /etc/nginx/sites-available/$DOMAIN.conf /etc/nginx/sites-enabled/$DOMAIN.conf

echo "Creating SSL directory"
mkdir -p /var/www/ssl/$DOMAIN/

echo "Restarting Nginx"
systemctl restart nginx

echo "Obtain certificate from LetsEncrypt"
certbot --nginx --redirect -d $DOMAIN

echo "Restarting Nginx"
systemctl restart nginx


```



Exit and save by pressing **Ctrl + x**, followed by **Y** and **Enter**




Finally run the script and follow instructions:




```bash
sh /root/add_server.sh
```



