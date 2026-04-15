---
id: 271
title: Ubuntu SSH + TOTP
type: post
date: 2020-08-31 09:14:35
lastmod: 2023-04-02 17:34:56
categories:
- Bash
---


In order to protect your SSH against hacks, the most basic you can do is to change the port, but that is never enough. We know the basics to be disable root logins, change port, enable pubkey authentication, IP based blocking and so on. However, one shoe never fits all. If you keep changing computers, it is very difficult to setup pubkey authentication on all computers, specially if you are not using your personal computer to login.

That is where the TOTP comes in. You still have to carry around a mobile phone, but I think we all do now a days.


### Install
Install google authenticator

```bash
sudo apt install libpam-google-authenticator
```


### Enable
and enable it for SSH:

```bash
sudo nano /etc/pam.d/sshd
```

Append to the bottom:
```bash
...
auth required pam_google_authenticator.so
```

And edit the ssh config file:

```bash
sudo nano /etc/ssh/sshd_config
```

to enable it in SSH config as well as change the SSH port

```bash
...
Port 15491 # change this to something random, preferably 5 digits
...
...
# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication yes # CHANGE THIS TO YES
...

```

### Setup TOTP for user
Now prepare your phone with your favorite TOTP app, like [google authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2) and run this command in shell:


```bash
google-authenticator
```

And answer as following:


```bash
Make tokens “time-base”": **yes
**
## SCAN THE QR CODE AT THIS STEP AND SAVE THE OFFLINE KEYS ##
Update the .google_authenticator file: **yes
**
Disallow multiple uses: **yes
**
Increase the original generation time limit: **no
**
Enable rate-limiting: **yes**
```


And then just restart ssh for the changes to take effect.


```bash
sudo systemctl restart sshd
```

Now use terminal on linux/mac or putty on windows to login. Do note that BitviseSSH does not work properly with this for some reason...



