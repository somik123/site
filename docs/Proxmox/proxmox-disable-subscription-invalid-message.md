---
id: 85
title: Proxmox disable subscription invalid message
type: post
date: 2019-10-20 08:57:24
lastmod: 2023-04-01 23:54:16
categories:
- Bash
---


When you are using the free version of Proxmox 5 or 6, you'll get a message every time saying your subscription is invalid. This is fine if it was just during the initial login, but it'll pop up every time you try to open a shell or run updates. Luckily this can be disabled by modifying one of the javascript files that Proxmox has.

!!! warning
    Only do it if you are running a private development server or to test out proxmox. For production use, purchase a license from Proxmox instead.


Note that you need to run this script EVERY time you upgrade Proxmox.


Go to your home directory and create the script, give it execution permission and edit it. You may need to install nano if you dont have it.


```bash
cd ~
touch proxmox_disable_message.sh
chmod +x proxmox_disable_message.sh
apt install nano
nano proxmox_disable_message.sh
```




```bash
#!/bin/sh

init_error() {
    local ret=1
    [ -z "$1" ] || printf "%s\n" "$1"
    [ -z "$2" ] || ret=$2
    exit $ret
}

# Original command
# sed -i.bak 's/NotFound/Active/g' /usr/share/perl5/PVE/API2/Subscription.pm && systemctl restart pveproxy.service

# Command to restart PVE Proxy and apply changes
PVEPXYRESTART='systemctl restart pveproxy.service'

# File/folder to be changed
TGTPATH='/usr/share/perl5/PVE/API2'
TGTFILE='Subscription.pm'

# Check dependecies
SEDBIN="$(which sed)"

[ -x "$SEDBIN" ] || init_error "Could not find 'sed' binary, aborting..."

# This will also create a .bak file with the original file contents
sed -i.bak 's/NotFound/Active/g' "$TGTPATH/$TGTFILE"
sed -i.bak 's/notfound/active/g' "$TGTPATH/$TGTFILE"
$PVEPXYRESTART

# Removed execution checking, since the terminal gets closed after PVEPXYRESTART anyways

exit 0
```



Save and exit by pressing **Ctrl+x** followed by **Y** and **Enter**.




Now you can run the script by simply typing in:



```bash
~/proxmox_disable_message.sh
```



You may need to force refresh your browser by holding Ctrl when you click the reload button for it to reload the new javascript. Or you can just close and open your browser (not tab, whole browser).



