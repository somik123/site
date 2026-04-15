---
id: 194
title: Ubuntu Nginx+PHP
type: post
date: 2019-11-13 14:25:06
lastmod: 2025-10-02 16:25:57
categories:
- Bash
---


This will install Nginx and PHP 7.x on Ubuntu or later and perform basic configurations needed to get it up and running. This is used to create servers to serve static content as well as minor PHP scripts that need to run on it.




After we'll set up SSL using LetsEncrypt and use it to transfer static and dynamic content over HTTPS.







Start by updating the server and installing Nginx as well as certbot:



```bash
sudo apt update
sudo apt upgrade
sudo apt install nginx nano sqlite3 python3-certbot-nginx
```



Then install PHP:




Only what's required




```bash
sudo apt install php-sqlite3 php-curl php-xml php-mbstring php-fpm php-gd php-imagick
```



Or full installation



```bash
sudo apt install php-cgi php-cli php-mysql php-gd php-imagick php-recode php-tidy php-xmlrpc php-sqlite3 php-curl php-xml php-mbstring php-fpm
```



Edit PHP.ini file:


```bash
sudo nano /etc/php/8.*/fpm/php.ini
```



And add these at the bottom:


```bash
short_open_tag = On
display_errors = on
error_reporting = E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED & ~E_WARNING
error_log = error_log
output_buffering = Off
date.timezone = "Asia/Singapore"
upload_max_filesize = 50M
post_max_size = 50M
memory_limit = 128M
```



Edit the nginx configuration file:


```bash
sudo nano /etc/nginx/nginx.conf 
```



And add this line to increase file upload size:

```bash
http {
  #...
  client_max_body_size 50M;
  #...
}
```






And enable PHP in Nginx site config files for your domain (where **example.com** is your domain)

```bash
sudo mkdir -p /var/www/example.com/public_html/
sudo touch /etc/nginx/sites-available/example.com.conf
sudo ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/example.com.conf 
```



Edit the example.com.conf file created in last step


```bash
sudo nano /etc/nginx/sites-available/example.com.conf
```



And edit it to make it look like this:





```bash
server {
        
        root /var/www/example.com/public_html/;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name example.com;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }


        listen 80;

}


```



Finally save and exit.




Now enable Nginx and PHP fpm to start on boot and run now.


```bash
sudo systemctl enable nginx php8.3-fpm
sudo systemctl restart php8.3-fpm
sudo systemctl restart nginx
```



Now lets get a certificate for the domain from certbot:




```bash
sudo certbot --nginx --redirect -d example.com
```



Follow the steps and you'll get a certificate.






