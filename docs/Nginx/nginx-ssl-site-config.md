---
id: 261
title: Nginx SSL Site Config
type: post
date: 2020-07-26 20:11:46
lastmod: 2023-04-01 23:59:45
categories:
- Bash
---


Sample configuration used for most SSL sites.





```bash
server {

        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name example.com;

        location / {
                        try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
               include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
               fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        }


        location ~ /\.ht {
                        deny all;
        }


    listen [::]:443 ssl; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        server_name example.com;

    listen 80;
    listen [::]:80 ;
    return 404; # managed by Certbot


}


```





