---
id: 26
title: Ubuntu Install Apache+PHP
type: post
date: 2019-10-08 16:56:59
lastmod: 2023-04-01 23:41:03
categories:
- Bash
---


This will install Apache2 and PHP 7.x along with LetsEncrypt to enable SSL protected site on Ubuntu or later and perform basic configurations needed to get it up and running.

## Update server
Always start by updating your server, followed by installing apache and openssl, finally enable SSL for apache:

```bash
sudo apt update
sudo apt upgrade
sudo apt install apache2 openssl
sudo a2enmod ssl
```


## Prepare
Now create a home directory for the domain you want to use with this server and create its configuration file:

```bash
sudo mkdir -p /var/www/domain.com/public_html
sudo chown -hR www-data:www-data /var/www/
sudo nano /etc/apache2/sites-available/domain.com.conf
```

And add to the config file:

```bash
<VirtualHost *:80>
    ServerName www.domain.com
    ServerAlias domain.com
    DocumentRoot /var/www/domain.com/public_html
    ErrorLog /var/www/domain.com/error.log
    CustomLog /var/www/domain.com/requests.log combined
</VirtualHost>
```

Save and exit using **Ctrl+x** followed by **Y** and then press **Enter**

Now enable it in apache and restart apache.

```bash
sudo ln -s /etc/apache2/sites-available/domain.com.conf /etc/apache2/sites-enabled/domain.com.conf

sudo apache2ctl configtest
sudo apache2ctl restart
```

## Add SSL certificates
Time to install certbot for getting a SSL certificate from LetsEncrypt. Once done, edit the apache config file for the domain.

```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt install python-certbot-apache

sudo certbot --apache -d domain.com -d www.domain.com

sudo nano /etc/apache2/sites-available/domain.com.conf
```


Append the SSL configurations to it:

```bash
<VirtualHost *:443>
    ServerName www.domain.com
    ServerAlias domain.com
    DocumentRoot /var/www/domain.com/public_html
    ErrorLog /var/www/domain.com/error.log
    CustomLog /var/www/domain.com/requests.log combined
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/domain.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/domain.com/privkey.pem
</VirtualHost>
```

Save and exit using **Ctrl+x** followed by **Y** and then press **Enter**

Finally restart apache and test your SSL enabled domain

```bash
sudo apache2ctl configtest
sudo apache2ctl restart
```


## Setup certificate renewal
Try a dry-run of the certificate renewal to confirm nothing has gone wrong:

```bash
sudo certbot renew --dry-run
```

If everything is ok, add it to cron:


```bash
sudo crontab -e
```
Append to make it run the renewal every 5 days:


```bash
0 0 */5 * * certbot renew
```


## Install PHP
Finally Install PHP 7.x, after it is installed, add it to apache.

```bash
sudo apt install php php-fpm php-cli php-common php-dev php-gd php-imap php-mbstring php-mysql php-pear php-snmp php-xml php-xmlrpc php-mcrypt

sudo apt install libapache2-mod-php7.0
sudo apache2ctl restart
```

Create a PHP test script to test your new configuration:

```bash
sudo echo '<?php phpinfo(); ?>' >> /var/www/domain.com/public_html/info.php
sudo chown -hR www-data:www-data /var/www/
```

You may want to edit your php.ini file


```bash
sudo nano /etc/php/7.x/cgi/php.ini
```


## Configure PHP 
To add the following at the bottom of it:

```bash
short_open_tag = On
display_errors = on
error_reporting = E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED
error_log = error_log
output_buffering = Off
date.timezone = "Asia/Singapore"
upload_max_filesize = 50M
post_max_size = 50M
```

Save and exit using **Ctrl+x** followed by **Y** and then press **Enter**

And restart apache to load the changes to php.ini

```bash
sudo apache2ctl restart
```


