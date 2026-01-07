---
title: 'Raspberry Pi VNC over SSH'
taxonomy:
    category:
        - docs
---

##  Raspberry Pi with no monitor

The document describes how to install a Raspberry Pi using Manjaro.

The end result will be GUI accesible by VNC over SSH.

The installation and configuration will be done entriely using SSH in a vritual terminal.

## Preparation

* RPi4
* Powersupply
* Network connection using a cable and a network switch
* 8GB SD card
* A cardreader

Download the minimal Manjaro Arm image - locate the latest version using [manjaro.org][1]

Unpack the compressed image using unxz

```bash
unxz -d Manjaro-ARM-minimal-rpi4-$YY.$MM.img.xz
```

As root Write the unpacked image to SD card

```bash
dd if=Manjaro-ARM-minimal-rpi4-$YY.$MM.img of=$DEVICE status=progress bs=4k conv=noerror,fdatasync oflag=dsync
```
Insert the SD card in your PI and connnect network cable and power.

When the PI has booted locate the pi using network utility like arp-scan.

As root run the command

```bash
arp-scan --local
```
Or you can use nmap to only look for host with port 22 active

```bash
nmap -p22 --open 192.168.x.0/24
```

If you only have one pi on the network it should be fairly easy to locate the IP address in the output e.g.

```text
...
192.168.x.y   dc:a6:32:xx:yy:yy   Raspberry Pi Trading Ltd
...
```
When you have located the device use ssh in terminal to connect to the pi

```bash
ssh root@192.168.x.y
```

When you are connected the OEM installer script will launch. Follow the prompts
and let the device restart and when restarted use ssh to reconnect - this time
using the username and password you created

## Update the system

Run pacman-mirrors and update the system

```bash
sudo pacman-mirrors --continent && sudo pacman -Syyu
```

Next thing is to install some GUI packages to be used when connecting using VNC.

## Xorg and drivers

This is a section I am unsure of. I have probably overdone the package selection but it is tested and it works

```bash
sudo pacman -Syu xorg-server xorg-server-common xorg-xinit
```

## LXDE

```bash
sudo pacman -Syu lxde epdfview accountsservice gnome-keyring gnome-icon-theme perl-file-mimeinfo xdg-user-dirs xdg-user-dirs-gtk xdg-utils
```

## Network utilities including VNC

```bash
sudo pacman -Syu tigervnc netctl ifplugd iw wpa_supplicant dialog network-manager-applet networkmanager-openvpn
```

## Setup and connect to VNC

Setup VNC on the Pi and connect as per this [linked topic][2]


[1]: https://manjaro.org/download/#ARM
[2]: https://forum.manjaro.org/t/root-tip-tigervnc-over-ssh/75087