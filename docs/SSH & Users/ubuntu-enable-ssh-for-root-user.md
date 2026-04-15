---
id: 37
title: Ubuntu enable SSH for root user
type: post
date: 2019-10-08 20:21:16
lastmod: 2023-04-01 23:42:54
categories:
- Bash
---

!!! danger
    This is a very bad idea for servers on open internet. Only recommended for servers on local network that are behind firewall and SSH is not accessible from internet. To be used for testing only.


### Install openssh-server
First install openssh server if not installed along with nano editor

```bash
sudo nano apt update
sudo nano apt upgrade
sudo apt install openssh-server nano
```

### Set root password
Change or set root user password

```bash
sudo passwd
```

### Enable root user login
Edit the ssh config file:

```bash
sudo nano /etc/ssh/sshd_config
```

Find the line that says:

```bash
#PermitRootLogin prohibit-password
```

And change it to:
```bash
PermitRootLogin yes
```

Save and exit using **Ctrl+x** followed by **Y** and then press **Enter**

Finally enable SSH during boot and restart it:

```bash
systemctl enable ssh
systemctl restart ssh
```

You can check the IP address of the server by running the command:


```bash
ip addr
```


