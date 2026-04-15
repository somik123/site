---
id: 90
title: Ubuntu Install LLMP stack
type: post
date: 2019-10-22 09:40:15
lastmod: 2023-04-01 23:56:21
categories:
- Bash
---


With the new MariaDB 10+ they have disabled performance_schema, which means your database will consume minimum amount of memory and you can run Lighttpd, PHP and MariaDB on only 256 MB RAM! So here's how to do it.




We'll also install phpMyAdmin for easy configuration of our mysql server and set it up securely to the best of our abilities.







Lets start by updating and installing Lighttpd and MariaDB as well as a few others.





```bash
sudo apt update
sudo apt upgrade
sudo apt install wget nano openssh-server unzip
sudo systemctl enable ssh
reboot
```



After reboot:





```bash
sudo apt install lighttpd mariadb-server
sudo systemctl start lighttpd.service
sudo systemctl enable lighttpd.service
```



Now install PHP





```bash
sudo apt-get install php-cgi php-cli php-mysql php-gd php-imagick php-recode php-tidy php-xmlrpc php-pdo php-sqlite3 php-curl php-xml libterm-readline-gnu-perl php-mbstring
```



And enable it in Lighttpd:





```bash
sudo lighttpd-enable-mod fastcgi
sudo lighttpd-enable-mod fastcgi-php
```



Now lets edit the PHP ini file:





```bash
sudo nano /etc/php/7.*/cgi/php.ini
```



And append these at the bottom of the file:





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



Now check what user Lighttpd is running as, as well as checking that the PHP was added:





```bash
sudo nano /etc/lighttpd/lighttpd.conf
```




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





Restart lighttpd to load new configs and enable PHP







```bash
sudo systemctl restart lighttpd.service
```





Create a PHP test script to test your new configuration:







```bash
sudo echo '<?php phpinfo(); ?>' >> /var/www/html/info.php
```



If everything is working fine until here, lets setup phpMyAdmin and MariaDB.




First lets startup and enable MariaDB





```bash
sudo systemctl restart mariadb
sudo systemctl enable mariadb
```



Now lets get phpMyAdmin, latest version, from their website <https://www.phpmyadmin.net/> and extract it.





```bash
cd /var/www/html/
sudo wget https://files.phpmyadmin.net/phpMyAdmin/4.9.1/phpMyAdmin-4.9.1-all-languages.zip
sudo unzip phpMyAdmin-*.zip
sudo mv phpMyAdmin-* phpMyAdmin
```



Now we need to edit the config file:





```bash
sudo cp /var/www/html/phpMyAdmin/config.sample.inc.php /var/www/html/phpMyAdmin/config.inc.php
sudo nano /var/www/html/phpMyAdmin/config.inc.php
```



And add some text to the blowfish secret:





```bash
...
...
/**
 * This is needed for cookie based authentication to encrypt password in
 * cookie. Needs to be 32 chars long.
 */
$cfg['blowfish_secret'] = 'ADD_SOME_$TRING_HERE'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
...
...
```



Then save and exit with **Ctrl+x** then **Y** then **Enter**.






Remember to set correct user for the folders and files under html:

```bash
sudo chown -hR www-data:www-data /var/www/html/
```



Now login to your MariaDB as root from console and add a new user, who will hold the same privileges as root. This is because you can no longer access the root account from phpMyAdmin on ubuntu unless your user has sudo privilages (unsafe).





```bash
sudo mysql -u root
```



Ensure performance_schema is disabled:





```bash
SHOW ENGINES;
```




```bash
+--------------------+---------+--------------------------------------------------------------------------------------------------+--------------+------+------------+
| Engine | Support | Comment | Transactions | XA | Savepoints |
+--------------------+---------+--------------------------------------------------------------------------------------------------+--------------+------+------------+
| MRG_MyISAM | YES | Collection of identical MyISAM tables | NO | NO | NO |
| CSV | YES | Stores tables as CSV files | NO | NO | NO |
| MEMORY | YES | Hash based, stored in memory, useful for temporary tables | NO | NO | NO |
| MyISAM | YES | Non-transactional engine with good performance and small data footprint | NO | NO | NO |
| SEQUENCE | YES | Generated tables filled with sequential values | YES | NO | YES |
| Aria | YES | Crash-safe tables with MyISAM heritage | NO | NO | NO |
| PERFORMANCE_SCHEMA | YES | Performance Schema | NO | NO | NO |
| InnoDB | DEFAULT | Percona-XtraDB, Supports transactions, row-level locking, foreign keys and encryption for tables | YES | YES | YES |
+--------------------+---------+--------------------------------------------------------------------------------------------------+--------------+------+------------+
```



First column says the Engine name, second column shows if it is supported, 4th column shows if it is disabled or not.




Ok, now lets create a new user with username "master" and password "secret_password". Remember to replace with your own username and password here:





```sql
CREATE USER 'master'@'localhost' IDENTIFIED BY 'secret_password';
```



Now give it full privileges and flush privileges:





```sql
GRANT ALL PRIVILEGES ON *.* TO 'master'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```



Finally restart mariadb for it to take effect





```bash
sudo systemctl restart mariadb
```



At this point, you can try to login with your defined username and password on phpMyAdmin. Everything is mostly done, except that your MariaDB is still unsecure. So lets secure it by running:





```bash
sudo mysql_secure_installation
```



By default, there is no root password, so press **Enter**.




Better to set a root password, so **Y**.




Then enter your desired MariaDB root password twice. It can be different from your server root password.




Best to remove anonymous users, so **Y**.




Disable all remote logins for root, **Y**.




Remove test database, **Y**.




Reload privilege table means whatever we did will take effect. If you made any mistakes, cancel it. If you feel everything is ok, **Y**.




Now restart MariaDB again:





```bash
sudo systemctl restart mariadb
```



And reboot your server for good measures. Once it is up, you are done.





```bash
sudo reboot
```


