---
title: 'Ubuntu Server on RPi'
date: '14:01 18-12-2022'
taxonomy:
    category:
        - docs
---

## Using rpi-imager

When using rpi-imager to prepare an sd-card with Ubuntu Server it is not clear how to finalize the setup.

After some triall'n'error I found the following.

Boot the image and wait a few minutes until the system settles - the green led is an excellent indicator.

When the system has settled either attach a monitor and keyboard or use nmap to locate the device

```
nmap -p 22 --open ip.x.y.z/24
```
The default login is **ubuntu:ubuntu** - you will immediately be required to change password, follow the prompts.