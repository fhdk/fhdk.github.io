---
title: 'Automated ISO login'
date: '07:09 05-03-2024'
taxonomy:
    category:
        - docs
---

## Automated login to TTY with Manjaro

Create a directory named getty@tty1.service.d/ inside the systemd system unit files directory:
```
sudo mkdir -p /etc/systemd/system/getty@tty1.service.d/
```
Create a file in this folder called override.conf with the following content:
```
# /etc/systemd/system/getty@tty1.service.d/override.conf
[Service]
ExecStart=
ExecStart=-/usr/bin/agetty --autologin YOUR_USERNAME_HERE --noclear %I $TERM
```
