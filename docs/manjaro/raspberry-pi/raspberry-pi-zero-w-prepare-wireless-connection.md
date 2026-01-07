---
title: 'Pi Zero W - prepare wireless connection'
date: '12:24 30-09-2022'
taxonomy:
    category:
        - docs
---

## RaspberryPi OS

This will likely work with most systemd based Linux distribution using systemd-networkd

Watch out for

- verify interface 
- with desktop dhcp will do
- [headless][2]

Prepare the SD card using RPI imager - mount the resulting root partition on the SD card.

[Raspberry Pi Documentation][3]

Edit the file as root

```
$MOUNTPOINT/etc/wpa_supplicant/wpa_supplicant.conf
```

Append the following and save the file

```
  ....
  
network={
    ssid="apname"
    psk="pskvalue or cleartext passphrase"
}
```
Edit the file as root
```
$MOUNTPOINT/etc/network/interfaces
```
Append the following and save the file
```
  ....

allow-hotplug wlan0

iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

```


__Source__ [Raspberry Pi wpa_cli][1]


[1]: https://www.raspberrypi.com/documentation/computers/configuration.html#using-the-command-line
[2]: https://www.raspberrypi.com/documentation/computers/configuration.html#setting-up-a-headless-raspberry-pi
[3]: https://www.raspberrypi.com/documentation/computers/configuration.html