---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Reinstall Manjaro without loosing data'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## BE WARNED
---
Even Gparted is a proven tool to do the job - the risk of losing data is always present when you work with the filesystem. 

**The author of this guide is not to be blamed if you mess up and loose data**.

**USING THIS GUIDE *IS ENTIRELY* YOUR RESPONSIBILITY**.

## Target audience
---
This guide has been written for a default Manjaro EFI based system using GPT partition scheme.

* Two partitions
   - $esp partition as first partition - **sdy1**
   - Single root partition as second partition - **sdy2**
* You have booted a live ISO
   - Several steps in the guide cannot be done if a partition is mounted.

The scope of this guide is not to cater for complex systems like dual-boot. You can of course amend the guide to suit your specific system. Don't hesitate to ask - after all that is what the community forum is all about.

## Using live ISO and Gparted
---
In this guide the device is assumed to be located at **/dev/sdy**. Usually it isn't and you need to locate the device you want to operate on. 

Use the terminal to locate you device path and replace **sdy** with the device name you find.

    $ lsblk

Switch to a root shell - on a live ISO the password is **manjaro**

    $ su -l root
    Password:
    [manjaro ~]#

## 1. File system check
---
If errors are found - do not proceed unless you have fixed them.
```
   # fsck /dev/sdy2
```
## 2. Backup
---
Now is the time to **backup important data** to external media - whether this is a physical or an online storage location.

## 3. Resizing
---
The Manjaro system itself do not require much space when **/home** is on a separate partition. A root partition of 32-64G is more than adequate.

### A. Shrink root partition
Release as much space as you can - preferably at least the space needed to hold the entire content of your current $USER folder.

### B. Create partition
Create a new partition using the claimed space and format to ext4 - it will become **sdy3**
```
    # mkfs.ext4 /dev/sdy3
```
### C. Temporary folders
Create two temporary folders e.g. `/mnt/home` and `/mnt/root`
```
    # mkdir -p /mnt/home
    # mkdir -p /mnt/root
```
### D. Mount the partitions
```
    # mount /dev/sdy2 /mnt/root
    # mount /dev/sdy3 /mnt/home
```
### E. Move data
Then move the folder `/mnt/root/your-user-name` to the new partition.
```
    # mv -r /mnt/root/home/your-user-name /mnt/home/
```
Depending on the amount of data and how much space you currently occupy - you may need to unmount the temporary mounts and repeat the shrinking of your primary partition.

Extend the new partition with the released space - remount and continue the moving of data.

**NOTE**: You can do a simple cut'n'paste within Thunar just remember to launch Thunar from the root shell. Remember your dot-files/folders - press <kbd>Ctrl</kbd>+<kbd>h</kbd> to toggle hidden files.

## 4. Install Manjaro
---
How you want to finish is up to you. Using the Calamares graphic installer presents two options.

**NOTE**:  Formatting of the $esp partition - not strictly required - but recommended.

### Option 1 
   - During install select the **Manual partition** option
   - Select the previous efi partition
     - mount point */boot/efi*
     - format using *FAT32*
     - ensure *$esp* and *boot* is selected
   - Select the previous root partition
     - Mount point */*
     - Format using *ext4*
   - Select the new partition
     - Mount point */home*
     - **do not format**
  - Continue the installer and reboot when done.

### Option 2
   - During install select the **Manual partition** option
   - Select the previous efi partition
     - mount point */boot/efi*
     - format using *FAT32*
     - ensure *$esp* and *boot* is selected
   - Select the previous root partition
     - Mount point */* 
     - format using *ext4*
   - Continue the install and reboot

#### Link your data 
When you have rebooted you can use the groundwork presented in the linked article from the #technical-issues-and-assistance:tutorials section

* https://archived.forum.manjaro.org/t/howto-move-your-personal-data-to-different-partition/47790?u=linux-aarhus

## Conclusion
---
If you reached this far without breaking sweat - you are a champion. :checkered_flag:
