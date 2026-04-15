---
id: 273
title: Ubuntu Add Sudo User
type: post
date: 2020-09-01 08:57:45
lastmod: 2023-04-02 00:01:27
categories:
- Bash
---

It is best not to use root user for anything as root is a common user name. It's better to create sudo users for these purposes.

Here's the easiest way to add a user to sudo group.

First create the user you want:


```bash
sudo adduser somik
```

```bash
Adding user 'somik' ...
Adding new group 'somik' (1000) ...
Adding new user 'somik' (1000) with group 'somik' ...
Creating home directory '/home/somik' ...
Copying files from '/etc/skel' ...
New password:
 **PASSWORD**
Retype new password:
 **PASSWORD**
passwd: password updated successfully
Changing the user information for somik
Enter the new value, or press ENTER for the default
    Full Name []:
    **NAME**
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n] **Y**
```

Then add the user to sudo group:


```bash
sudo usermod -aG sudo somik
```

That's it. Now login as the user.



