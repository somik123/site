---
id: 72
title: Ubuntu Install Aria2c for Downloads with WebUI
type: post
date: 2019-10-15 18:48:29
lastmod: 2023-04-01 23:51:18
categories:
- Bash
---


This is a simple guide on how to install and configure Aria2c with WebUI for downloading files on your linux server. 




For the WebUI, we'll go with lighttpd as it is lightweight, however you can install apache or nginx or even run the file directly from your computer.







Lets start by updating the server, then installing and enabling the lighttpd module as well as a few other modules.





```bash
sudo apt update
sudo apt upgrade
sudo apt install lighttpd make wget nano screen unzip
sudo systemctl start lighttpd.service
sudo systemctl enable lighttpd.service
```



Now lets get the latest static build of Aria2 OpenSSL from the build site: <https://github.com/q3aql/aria2-static-builds/releases>




You can also get it from our mirror: [aria2-1.35.0-linux-gnu-64bit-build1.tar.bz2](https://somik.org/get/7f131253092d89f7744627aa1e0f5f040a638b8176c97be8dda6c14748b85780/aria2-1.35.0-linux-gnu-64bit-build1.tar.bz2)




Now transfer it to your home directory. If you would rather download it directly to your directory, do this (with the link you want). Then extract and install it.





```bash
wget https://somik.org/get/7f131253092d89f7744627aa1e0f5f040a638b8176c97be8dda6c14748b85780/aria2-1.35.0-linux-gnu-64bit-build1.tar.bz2
tar jxvf aria2-*.tar.bz2
cd aria2-*
sudo make install
```



Now create a Aria2 downloads directory and configuration file:





```bash
mkdir -p /home/somik/Downloads/
nano ~/aria2.conf
```




```bash
dir=/home/somik/Downloads/
max-concurrent-downloads=10
seed-ratio=1
max-connection-per-server=16
min-split-size=50M
rpc-save-upload-metadata
enable-rpc=true
rpc-listen-all=true
rpc-secret=AUTHENTICATION_KEY_GOES_HERE
rpc-listen-port=6800
check-certificate=false
```



Here you can set the default downloads directory, authentication token, and various other values. Be sure to set "*check-certficate*" to **false** if you are using Letsencrypt as it does not recognize Letsencrypt as of now.




Finally create a startup script for aria:





```bash
touch ~/start_aria2.sh
sudo chmod +x ~/start_aria2.sh
nano ~/start_aria2.sh
```




```bash
#!/bin/bash
screen -dmS scr_00 bash -c "aria2c --conf-path=/home/somik/aria2.conf"
```



Finally add it to cron so it starts on boot:





```bash
crontab -e
```



And append:





```bash
@reboot /home/somik/start_aria2.sh
```



Now, lets get the WebUI setup. Download the WebUI from: <https://github.com/ziahamza/webui-aria2/> 




 You can also get it from our mirror: [aria2c-webui.zip](https://somik.org/get/9c40a639b16c599da78c88a675940f40eddb5108511c7a72f391580f783b9941/aria2c-webui.zip)




Upload it to your */var/www/html* folder:





```bash
cd /var/www/html
sudo wget https://somik.org/get/9c40a639b16c599da78c88a675940f40eddb5108511c7a72f391580f783b9941/aria2c-webui.zip
unzip aria2*
sudo chown -hR www-data:www-data /var/www/html
```



Edit the app.js and add your authentication token for easy access.





```bash
sudo nano /var/www/html/app.js
```



Find the line 342, which should say something like "*(c = p && p.auth && p.auth.token ? p.auth.token : null),*":





```js
...
...
                if (g && u.length)
                  return (
                    (g = !1),
                    (p = u[0]),
                    (c = p && p.auth && p.auth.token ? p.auth.token : null),
                    e.init(p),
                    void (h = setTimeout(v, t))
                  );
...
...
```



And replace it with your authentication key in double quotes:





```js
...
...
               if (g && u.length)
                  return (
                    (g = !1),
                    (p = u[0]),
                    (c = "AUTHENTICATION_KEY_GOES_HERE"),
                    e.init(p),
                    void (h = setTimeout(v, t))
                  );
...
...
```



Now save and exit. Finally restart your server:





```bash
sudo reboot
```



That's it. Now you can access Aria2 and its WebUI from your server.



