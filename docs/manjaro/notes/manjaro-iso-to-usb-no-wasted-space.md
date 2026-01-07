---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'ISO to USB no wasted space'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Manjaro ISO
From time to time the question is raised on how an ISO is written to USB - especially large capacity USB.

> Why can't I use the remaining space on my 32GB USB?

This is because Manjaro ISO is a ISO9660 cd-rom file system and such filesystem is immutable - you cannot change it. As such a 2GB ISO makes it impossible to use the excess space for anything - it just sits there unused.

So what can we do?

### The choices
1. ventoy
2. Manjaro on-a-stick
3. Multi Boot USB

## 1. ventoy
[ventoy] is a set of small binaries compiled from various open source projects - most notably grub and exfat.

It is also available for Windows from the [ventoy] web

It installs bootloader for efi and mbr and formats the remaining part of the device using exfat.

Copy your ISO files to the partition and boot the your system from the stick. ventoy locates all ISO files and presents them the same manner as below script.

Then you select the ISO to boot. It is even simpler than below method.

Install the package from the repo

    pamac install ventoy

When done run the script to display your options

    ventoy

Locate your usb stick

    lsblk

Then initialize the stick (one time operation)

    sudo ventoy -i /dev/sdy

When done the disk is available in your file manager - copy your ISO files to the stick.

:exclamation: Eject the stick using the eject button in your file manager and wait. Depending on the number of ISO, the speed and quality of your USB disk it will take a long time to complete. Have patience ... patience - only when the USB disappears from your file manager you can remove it.

You can use this method for Windows ISO too

* https://archived.forum.manjaro.org/t/howto-use-manjaro-to-create-a-bootable-windows-usb/92780

## 2. Manjaro on a stick
One way of doing this is creating a Manjaro installation on USB with an added script for booting ISO directly from Grub.

* https://archived.forum.manjaro.org/t/howto-run-manjaro-on-a-stick/109520?u=linux-aarhus

Later I will describe a stand-alone solution based on the same script used in above article.


## 3. Multi Boot USB

* https://mbusb.aguslr.com/

With the script it is insanely easy to create a stick that boots a variety of ISOs. And Arch based ISOs work extremely well - including Manjaro.

You can download the collection of scripts as an archive or your can clone from Github - what ever you prefer. This guide uses git.

Install git - if you don't have it
```bash
❯ sudo pacman -Syu git
```
Clone the repo
```bash
❯ git clone https://github.com/aguslr/multibootusb
Cloning into 'multibootusb'...
...
```
Navigate into the repo folder
```bash
❯ cd multibootusb
```
To get an idea of what the script is doing
```bash
❯ ./makeUSB.sh -h
Script to prepare multiboot USB drive
Usage: makeUSB.sh [options] device [fs-type] [data-size]

 device                         Device to modify (e.g. /dev/sdb)
 fs-type                        Filesystem type for the data partition [ext3|ext4|vfat|ntfs]
 data-size                      Data partition size (e.g. 5G)
  -b,  --hybrid                 Create a hybrid MBR
  -c,  --clone                  Clone Git repository on the device
  -e,  --efi                    Enable EFI compatibility
  -i,  --interactive            Launch gdisk to create a hybrid MBR
  -h,  --help                   Display this message
  -s,  --subdirectory <NAME>    Specify a data subdirectory (default: "boot")
```
The default **fs-type** is **vfat**. 

The **data-size** is not defined. The **data-size** option is only useful if you want to set aside space for a persistent storage partition. The storage space for ISO files is then limited to the specified size of e.g. 5G.

When you use the **data-size** option - the remainder of the USB is left untouched. To be able to actually use the space you will need to manually create a partition using the remaining space. One example is a LUKS encrypted container for your sensitive data.

Remove USB and list devices
```bash
❯ lsblk -la 
```
Insert USB and do it again - note the added device and create the USB - using **vfat** makes the device visible in Windows too.

The following commands will assume your device is **`/dev/sdy`** - replace the device name with the device name from your system.

**NOTE**: The script prints each command to the console before executing - so you can follow what the script is currently doing.

The following creates a hybrid USB using vfat
```bash
❯ sudo ./makeUSB.sh -b -e /dev/sdy vfat
```
To create an USB only for Linux use ext4
```bash
❯ sudo ./makeUSB.sh -b -e /dev/sdy ext4
```


Download a Manjaro ISO, open the device in your file manager and copy a Manjaro ISO to the folder **/boot/isos** on the USB. Use the eject function of your file manager. You can also do it from terminal.

**IMPORTANT**: Do not remove the device until all buffers are flushed to disk. If you want to make sure the ISO is written correct - checksum verify the resulting ISO on the USB stick.

```bash
❯ mkdir ~/usb-multiboot
❯ sudo mount /dev/sdy3 ~/usb-multiboot
❯ cp ~/Downloads/manjaro*.iso ~/usb-multiboot/boot/isos
❯ sync
❯ sudo umount ~/usb-multiboot
❯ sync
```

- Boot your system from the USB
- Select **Multiboot >**
- Wait while configuration is read
- The more ISOs the longer read time

## Tips
The stick presumably supports more than 100 different distributions - so the **Multiboot >** menu takes a while to load - because of the dynamic nature of including the ISOs.

To speed up the initial USB Multiboot configuration - you can remove folders from **/boot/grub/mbusb.d/** you don't intend to use.

## Issues
If you encounter any issues with the script or the configuration files for booting an ISO - please create an issue at the project's [Github](https://github.com/aguslr/multibootusb) page.

## Revisions
[date=2020-05-09 time=09:15:00 timezone="Europe/Copenhagen"]
* added ventoy


[ventoy]: https://ventoy.net
