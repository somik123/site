---
id: 62
title: Ubuntu Install Pi-Hole with PiVPN
type: post
date: 2019-10-14 20:48:24
lastmod: 2023-04-01 23:53:20
categories:
- Bash
---


This will transform your server into a network wide ad-blocker as well as allowing you to use this ad-blocker from outside your network using the private VPN of PiVPN. These software are originally meant to be installed on a Raspberry Pi, but since you got a space on your server, why not install it? This is the easiest way to get OpenVPN server installed on your server. Did I mention that it'll also block ads?







Do take note that some of the commands will be used without "sudo" while some are using it.




So as always, start by updating your server and installing required components:





```bash
sudo apt update
sudo apt upgrade
sudo apt install curl 
```



You can install either Pi-Hole or PiVPN first since we are going to install both and configure it in a way that will make them talk to each-other. On this occation, we'll install PiVPN first.




Install PiVPN
-------------





```bash
curl -L https://install.pivpn.io | bash
```



The OS is untested? No issues, it'll work fine. Click **Yes**.




Installer starting, click **OK** followed by **OK**, **OK**, **OK** until you arrive at the user selection screen.




Choose a user (the username you logged in as in your ubuntu other then root) followed by **OK**.




Ports need to open? **OK**.




Unattended security patches? Better to click **Yes**.




Protocol **UDP** is better as it is hidden from port scans, followed by **OK**.




Choose a custom port. Recommendations to change it to anything other then 1194, the default port. Best to choose a port number between 10,000 and 65,000.




Stronger encryption using OpenVPN 2.4? **Yes** please.




Encryption level of **256** is enough but you can choose higher since your server is more powerful then a Pi.




Now, if you are using a fixed IP, you can choose it. In our case, we will choose a **DNS entry** as it will allow us to connect even if the IP changes.




If you selected DNS entry, enter the dns entry here like: myserver.example.com




Confirm the setting is correct by choosing **Yes**.




Choose a DNS provider here. You can choose anything as we will be changing it later. In our case, we'll go with Cloudflare as it has a easy to recognize IP.




Do you need a custom domain? Choose **No** here.




And we are done! Click **Ok**.




Choose **Yes** followed by **Ok** to reboot.




Now it is time to install Pi-Hole.




Install Pi-Hole
---------------





```bash
curl -sSL https://install.pi-hole.net | bash
```



Start the installer with **Ok**, **Ok**, and **Ok** again.




Time to choose an interface. Choose the first one that is NOT *tun0*. We'll add it later in settings.




Choose a DNS provider you prefer for Pi-Hole to check and update it's DNS records.




Default adblock list is fine for this step.




IPv4 and IPv6, choose the ones you need. Normally keeping both ON isn't any issue.




Will you use existing network settings as static IP? **Yes** then **Ok**.




Do you need the admin web interface? Yes, so make sure **On** is checked before **Ok**.




Do you want to install lighttpd for the web interface? If you do not have apache or nginx installed or are not planning to installing them later, better to choose **On** and then **Ok**.




Log queries? Better to leave it **On** for troubleshooting.




**Show everything**, including passed and blocked domains and IPs that requested it? **Yes**.




Now let the installer do it's thing before the completed screen with the confusing **password** appears.




First thing to do, change admin password...





```bash
pihole -a -p
```



Now you can access pi-hole admin interface on your IP or domain to check by going to http://192.168.xx.xxx/admin/ or http://myserver.example.com/admin/




Now lets make them talk to eachother...




Pi-Hole & PiVPN configuration
-----------------------------





```bash
sudo nano /etc/openvpn/server.conf
```



Find the lines with your DNS provider IP address (1.1.1.1 and 1.0.0.1 in our case since we went with Cloudflare).





```bash
...
# Set your primary domain name server address for clients
push "dhcp-option DNS 1.1.1.1"
push "dhcp-option DNS 1.0.0.1"
...
```



and replace with





```bash
...
# Set your primary domain name server address for clients
push "dhcp-option DNS 10.8.0.1"
...
```



Then save and exit with **Ctrl+x** the **Y** then **Enter**.




Next edit the Pi-Hole configuration files:





```bash
sudo nano /etc/pihole/setupVars.conf
```



And add the line "*PIHOLE_INTERFACE=tun0*" bellow the existing "*PIHOLE_INTERFACE=*" line. Make sure it looks something similar to this.





```bash
PIHOLE_INTERFACE=ens18
PIHOLE_INTERFACE=tun0
...
...
```



The *ens18* can be *eth0* or something else depending on what your ethernet module is called. The tun0 must be added after that.






Then save and exit with **Ctrl+x** the **Y** then **Enter**.






Finally create a new file under dnsmasq to add the openvpn interface.




```bash
sudo nano /etc/dnsmasq.d/02-ovpn.conf
```



Add:





```bash
interface=tun0
```



Then save and exit with **Ctrl+x** the **Y** then **Enter**. 




Wait, you needed to add local domains to Pi-Hole? Lets do it now.





```bash
sudo touch /etc/hosts.mydomain
sudo nano /etc/dnsmasq.d/02-ovpn.conf
```



Append:





```bash
...
addn-hosts=/etc/hosts.mydomain
```



Then save and exit with **Ctrl+x** the **Y** then **Enter**.




Now go back and edit the hosts file we created:





```bash
sudo nano /etc/hosts.mydomain
```



And add your domains, one per line, IP followed by domain name like so:





```bash
192.168.1.10 mydomain.com
192.168.1.11 private.example.com
```



Finally reboot your server:





```bash
sudo reboot
```



Final configurations
--------------------




If you want to add users to your PiVPN, you can use the command:





```bash
pivpn add
```



All of your new OpenVPN configuration files will be saved at /home/username/ovpns/ directory.




You may want to configure additional blocklists for Pi-Hole:




<https://discourse.pi-hole.net/t/update-the-best-blocking-lists-for-the-pi-hole-alternative-dns-servers-2019/13620>




Start by changing to pihole directory and editing the adlist file:





```bash
sudo nano /etc/pihole/adlists.list
```



Edit it to make it look like so (be sure to remove any tabs or spaces from the links)





```text
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://mirror1.malwaredomains.com/files/justdomains
http://sysctl.org/cameleon/hosts
https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_ad.txt


https://raw.githubusercontent.com/hectorm/hmirror/master/data/adaway.org/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/adblock-nocoin-list/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/adguard-simplified/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/anudeepnd-adservers/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/disconnect.me-ad/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/disconnect.me-malvertising/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/disconnect.me-malware/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/disconnect.me-tracking/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/easylist/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/easyprivacy/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/eth-phishing-detect/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/fademind-add.2o7net/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/fademind-add.dead/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/fademind-add.risk/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/fademind-add.spam/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/kadhosts/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/malwaredomainlist.com/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/malwaredomains.com-immortaldomains/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/malwaredomains.com-justdomains/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/matomo.org-spammers/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/mitchellkrogza-badd-boyz-hosts/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/pgl.yoyo.org/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/ransomwaretracker.abuse.ch/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/someonewhocares.org/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/spam404.com/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/stevenblack/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/winhelp2002.mvps.org/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/zerodot1-coinblockerlists-browser/list.txt
https://raw.githubusercontent.com/hectorm/hmirror/master/data/zeustracker.abuse.ch/list.txt
https://raw.githubusercontent.com/CHEF-KOCH/Audio-fingerprint-pages/master/AudioFp.txt
https://raw.githubusercontent.com/CHEF-KOCH/Canvas-fingerprinting-pages/master/Canvas.txt
https://raw.githubusercontent.com/CHEF-KOCH/WebRTC-tracking/master/WebRTC.txt
https://raw.githubusercontent.com/CHEF-KOCH/CKs-FilterList/master/Anti-Corp/hosts/NSABlocklist.txt
https://gitlab.com/quidsup/notrack-blocklists/raw/master/notrack-blocklist.txt
https://gitlab.com/quidsup/notrack-blocklists/raw/master/notrack-malware.txt
https://www.stopforumspam.com/downloads/toxic_domains_whole.txt
```





Then save and exit with **Ctrl+x** the **Y** then **Enter**.






Also download the whitelist to this folder.





```bash
wget https://raw.githubusercontent.com/raghavdua1995/DNSlock-PiHole-whitelist/master/whitelist.list
sudo cp whitelist.list /etc/pihole/whitelist.txt
```



Then update gravity.





```bash
pihole -g
```



Ensure everything is OK by looking at the green ticks.




Finally change your DNS to PiHole IP or start using PiVPN.



