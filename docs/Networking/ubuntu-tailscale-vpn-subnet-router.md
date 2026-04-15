---
id: 295
title: Ubuntu Tailscale VPN + Subnet router
type: post
date: 2023-04-01 23:16:11
lastmod: 2023-04-01 23:29:22
categories:
- Bash
---


This is a step by step guide to install Tailscale VPN and setup Subnet router on Ubuntu OS. The guide will allow installation on both physical hardware, KVM virtual machine, as well as unprivileged LXC containers.







Ignore the following part if not using LXC containers. Edit the LXC container config file from the host machine. In this example, lets say the LXC container ID is 102.





```bash
nano /etc/pve/lxc/102.conf
```



Add at the bottom of the config file:





```bash
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```



This step onward is applicable for all.




Run the following in the machine you are installing Tailscale on to setup the subnets:





```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```



Then install tailscale with the following command:





```bash
curl -fsSL https://tailscale.com/install.sh | sh
```



Bring up the tailscale with the subnet:





```bash
sudo tailscale up --advertise-routes=10.0.0.0/24,192.168.1.0/24
```



Copy the url provided into your browser to login and edit the device to approve the subnet router request.




That's it.



