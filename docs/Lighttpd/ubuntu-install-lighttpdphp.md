---
id: 36
title: Ubuntu Install Lighttpd+PHP
type: post
date: 2019-10-08 20:54:02
lastmod: 2023-04-01 23:45:04
categories:
- Bash
---


This will install Lighttpd and PHP 7.x on Ubuntu or later and perform basic configurations needed to get it up and running. This is used to create isolated dev servers that use as little memory as possible. 




You can also setup SSL on these using LetsEncrypt, but it is not covered here. This is to be used for dev or testing or used on local network with a Nginx proxy handling SSL.







Start by updating the server and installing lighttpd:





```bash
sudo apt update
sudo apt upgrade
sudo apt install lighttpd
```



Enable lighttpd to run on boot and start it up now:





```bash
sudo systemctl start lighttpd.service
sudo systemctl enable lighttpd.service
```



Install PHP as well as "readline" perl module required for fastcgi





```bash
sudo apt-get install php-cgi php-cli php-mysql php-gd php-imagick php-mbstring php-recode php-tidy php-xmlrpc php-pdo php-sqlite3 php-curl php-xml libterm-readline-gnu-perl
```



Now install and enable PHP mod for lighttpd





```bash
sudo lighttpd-enable-mod fastcgi 
sudo lighttpd-enable-mod fastcgi-php
```



Edit the php.ini file:





```bash
sudo nano /etc/php/7.*/cgi/php.ini
```



And append the following at the bottom of the file:





```bash
short_open_tag = On
display_errors = on
error_reporting = E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED
error_log = error_log
output_buffering = Off
date.timezone = "Asia/Singapore"
upload_max_filesize = 50M
post_max_size = 50M
memory_limit = 128M
```



Now edit the lighttpd config file:





```bash
sudo nano /etc/lighttpd/lighttpd.conf
```



Take note of the following lines:





```bash
...
...
server.document-root = "/var/www/html"
...
...
server.username = "www-data"
server.groupname = "www-data"
server.port = 80


index-file.names = ( "index.php", "index.html", "index.lighttpd.html" )
...
static-file.exclude-extensions = ( ".php", ".pl", ".fcgi" )
...
...

```



This shows our server document root, the user lighttpd is running as and the fact that PHP has been added correctly.




Next check the php fastcgi server configuration file:




Ensure it looks similar to this:





```bash
sudo nano /etc/lighttpd/conf-available/15-fastcgi-php.conf
```




```bash
# -*- depends: fastcgi -*-
# /usr/share/doc/lighttpd/fastcgi.txt.gz
# http://redmine.lighttpd.net/projects/lighttpd/wiki/Docs:ConfigurationOptions#mod_fastcgi-fastcgi

## Start an FastCGI server for php (needs the php5-cgi package)
fastcgi.server += ( ".php" =>
    ((
        "bin-path" => "/usr/bin/php-cgi",
        "socket" => "/var/run/lighttpd/php.socket",
        "max-procs" => 1,
        "bin-environment" => (
                "PHP_FCGI_CHILDREN" => "4",
                "PHP_FCGI_MAX_REQUESTS" => "10000"
        ),
        "bin-copy-environment" => (
                "PATH", "SHELL", "USER"
        ),
        "broken-scriptfilename" => "enable"
    ))
)
```



Restart lighttpd to load new configs and enable PHP





```bash
sudo systemctl restart lighttpd.service
```



Create a PHP test script to test your new configuration: 





```bash
sudo echo '<?php phpinfo(); ?>' >> /var/www/html/info.php
```



Remember to set correct user for the folders and files under html:





```bash
sudo chown -hR www-data:www-data /var/www/html/
```


