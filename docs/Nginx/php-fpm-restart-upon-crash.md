---
id: 280
title: PHP-FPM Restart Upon Crash
type: post
date: 2021-02-26 09:32:38
lastmod: 2023-04-02 00:03:05
categories:
- Bash
- PHP
---


PHP-FPM service, when running as socket, seems to crash making Nginx output errors and users frustated. This is a simple fix to bypass the issue with auto restarting the php upon crash. Although it is recommended to see why the crash occurs in first place and fix that first.







Edit the PHP FPM configuration file:





```bash
sudo nano /etc/php/7.*/fpm/php-fpm.conf
```



Find and change the following lines as such





```bash
...
...
...

; If this number of child processes exit with SIGSEGV or SIGBUS within the time
; interval set by emergency_restart_interval then FPM will restart. A value
; of '0' means 'Off'.
; Default Value: 0
emergency_restart_threshold = 10

; Interval of time used by emergency_restart_interval to determine when
; a graceful restart will be initiated. This can be useful to work around
; accidental corruptions in an accelerator's shared memory.
; Available Units: s(econds), m(inutes), h(ours), or d(ays)
; Default Unit: seconds
; Default Value: 0
emergency_restart_interval = 60

; Time limit for child processes to wait for a reaction on signals from master.
; Available units: s(econds), m(inutes), h(ours), or d(ays)
; Default Unit: seconds
; Default Value: 0
process_control_timeout = 10s

...
...
...
```





