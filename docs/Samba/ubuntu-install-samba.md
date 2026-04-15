---
id: 98
title: Ubuntu Install Samba
type: post
date: 2019-10-31 15:03:45
lastmod: 2025-10-02 16:26:12
categories:
- Bash
---


So you want to use a linux server on local network. Great, but if you still have to access the files through ssh or ftp, it is troublesome. So why not access the files over samba network share? It is also great when you want to setup a simple NAS server on your local network.



### Install
As always, first step, update server and install samba:

```bash
sudo apt update
sudo apt upgrade
sudo apt install samba
```


### Configure
Now we need to add a linux user who will hold the credentials. In this case, we'll add a user "somik" so remember to change it to what you need:

```bash
sudo adduser somik
```

You should see something similar to this. Enter password any details you want to. Only password is required:

```bash
Adding user `somik' ...
Adding new group `somik' (1001) ...
Adding new user `somik' (1001) with group `somik' ...
Creating home directory `/home/somik' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for somik
Enter the new value, or press ENTER for the default
    Full Name []:
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n] Y
```

Once done, add the user we added above to Samba, then enter password when asked to:


```bash
sudo smbpasswd -a somik
```

Now edit the samba config file:

```bash
sudo nano /etc/samba/smb.conf
```

Take note of the following:


```bash
...
...
workgroup = WORKGROUP
...
...
```

And append this at the bottom of the file, where "DocuRoot" is the display name of the shared folder, "*/var/www/html*" is the folder we want to share (can be different in your case, just create it beforehand), and "www-data" is the user and group of the folder.


### Setup shares

```bash
...
...
[DocuRoot]
    delete readonly = yes
    writable = yes
    path = /var/www/html
    force directory mode = 755
    force group = www-data
    force create mode = 644
    force user = www-data
    comment = DocuRoot
    create mode = 0644
    public = no
    guest ok = no
    browsable = yes
    directory mode = 0755
    veto files = /._*/.DS_Store/
    delete veto files = yes
```




And test your samba config file with


```bash
 sudo testparm
```

If no errors, start up your samba server and enable it on boot. Then just access it like any network share from your computer.

```bash
sudo systemctl enable smbd nmbd
sudo systemctl start smbd nmbd
```


