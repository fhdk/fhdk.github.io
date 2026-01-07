---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Encrypted CLI installation'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Encrypt a new Manjaro installation using CLI
---
This guide is an addition to the CLI guide on creating a Manjaro installation.

The guide will assume a few points

* You are installing on bare-metal.
* Your disk device is **/dev/sda**
* If you device **is not** the above - you **must** replace **sda** with your actual device.
* Target is an EFI based system.
* You are familiar with the CLI guide.

**DISCLAIMER**: I assume no responsiblity for your device and the consequenses of your actions on your device. Previous to this guide I had **no** experience in creating encrypted partition - let alone encryption at all. If you think there may be an error in the guide please help me improve by adding a comment.

I have had no need for encryption but learning is good and my learning starts with a search using [DuckDuckGo](https://duckduckgo.com/?q=install+encrypted+arch+linux).

This guide will **only** cover the steps to create an encrypted system. When you have created the required encrypted partitions the installation follows the same pattern as the previous guide. You find the link at then end of this document.

## Overview
1. Preparation
2. Partitioning
3. Formatting
4. Mounting
5. Base install
6. Bootloader
7. Conclusion


## Preparation
---
This step is going to take some time - depending your level of motivation, your actual need or if it is just for the learning experience.

When a disk is used the data written to the disk can be read with forensic utilities such as the testdisk utility.

* https://gitlab.com/cryptsetup/cryptsetup/wikis/FrequentlyAskedQuestions#2-setup

When you re-purpose a disk using a partitioning tool, you are removing references to files but not the files themselves.  Even in the case of creating encrypted partitions the previous raw data is still present.

If you want to create encrypted partitions - the whole purpose is to hide whatever data you want hide but in case of re-using a disk you also want to remove every shred of data previously present.

### Possible ways to shred data on a disk
Several possible ways exist for shredding data and the chosen method is a matter of choice.

The choice depends on multiple factor like purpose, security needs, time available or if it just a learning experience. If you are using an SSD it could be a good idea to use a tool to reset the memory cells.

* https://wiki.archlinux.org/index.php/Solid_state_drive/Memory_cell_clearing

#### Shred for giveaway using dd
Shredding of existing data is most easily done overwriting the disk with zero's (0) a number of times. Executing this command as root a number of times will do it.

    # dd if=/dev/zero of=/dev/sda bs=1M status=progress

That is good if you are giving the disk away, but it is not good enough if you re-purpose the disk for your own use as an encrypted device.

If you only overwrite your disk with zero's any data written to the encrypted partition will be obvious to a forensic tool because the data is everything but zero's. This is why re-using a disk for encrypting needs a little more sophistication.

#### Shred for encryption using dd
Overwrite the disk a number of times using random sequence is the preparation - even for a brand new disk.

    # dd if=/dev/urandom of=/dev/sda bs=1M status=progress

This preparation is not required but if you are concerned of the risks of anyone using forensic tools on your disk - you will do it.

#### Shred for encryption using shred
The shred utility is working slightly different, even as it operates on the device path, it uses the file allocation table to identify files, then overwrite the file a number of times (defaults to three (3)).

#### Arch wiki
The wiki describes a procedure to wipe a device

* https://wiki.archlinux.org/index.php/Dm-crypt/Drive_preparation

## Partitioning
---
We will use **cfdisk** to partition the disk. Select **gpt** when prompted.

    # cfdisk --zero /dev/sda

We need a minimum of three (3) partitions. If you plan to use multiple kernels the boot partition should be big enough to hold all of them. My initial 100M would be too small as pointed to by @eugen-b so I changed to 300M.
 
* $esp
  * size: 100M
  * partition type: EFI system
* boot
  * size: 300M
  * partition type: Linux
* root
  * size: what you have left

When you have created the partitions - write the changes and exit cfdisk.

### Encrypted device
Now we create our encrypted device

    # cryptsetup --verbose --hash sha512 --iter-time 5000 --use-random luksFormat /dev/sda3

Confirm the creation and supply a passphrase - the longer the better - use a combination of upper/lowercase letters, numbers and symbols.

    WARNING!
    ========
    This will overwrite data on /dev/sda3 irrevocably.

    Are you sure? (Type uppercase yes): YES
    Enter passphrase for /dev/sda3:

**IMPORTANT**: If you are using Dvorak or other non-standard read the forum topic linked below on custom keymap. Usually you can use a passphrase that can be reproduced using a standard qwerty US keyboard layout. 

    Verify passphrase:
    Key slot 0 created.
    Command successful.

### Unlock the partition
Now create a device mapper by unlocking your partition - in this case we use the name **cryptroot** but it can really be anything - just stick to it because you will need it later.

    # cryptsetup open --type luks /dev/sda3 cryptroot
    Enter passphrase for /dev/sda3:

## Formatting
---
The $esp partition uses FAT32

    # mkfs.fat -F32 /dev/sda1

The boot partition uses ext - ext2 will be just fine

    # mkfs.ext2 /dev/sda2

The encrypted partition

    # mkfs.ext4 /dev/mapper/cryptroot

## Mounting
---
Mount the devices so we can begin installing

### Mount root

    # mount /dev/mapper/cryptroot /mnt

### Create boot folder and mount

    # mkdir /mnt/boot
    # mount /dev/sda2 /mnt/boot

### Create efi folder and mount

    # mkdir /mnt/boot/efi
    # mount /dev/sda1 /mnt/boot/efi

### Create etc folder

    # mkdir /mnt/etc

### Generate filesystem table

    # fstabgen -U /mnt > /mnt/etc/fstab

## Base installation
---
Enter a chroot environment and use **section 6 and 7** of the CLI guide to perform a basic Manjaro installation.

    # manjaro-chroot /mnt /bin/bash

* https://forum.manjaro.org/t/howto-install-manjaro-using-cli-only/108203?u=linux-aarhusaarhus

Return to this guide when done and don't leave the chroot.

## Bootloader
---
To enable grub to use the encrypted root we need to make some changes to some default configs.

### Edit mkinitcpio.conf
Why?

To my understanding of initramfs on Arch it is generated partly using predefined modules and partly on modules detected at boot time.

If you need a particular keyboard or keymap available as early as possible - inputting passphrase comes to mind - you need make them explicit available. 

That means changing the HOOKS line to make the modules available before autodetect. This makes your initramfs image larger - presumably - I have no means of testing - it also makes your system aware of your preferences (e.g. a dvorak keyboard or maybe even country specific layout)

This is a point I could use some feedback.

>```
> # nano /etc/mkinitcpio.conf
> ```
>```
>HOOKS="base udev block keyboard keymap autodetect modconf encrypt filesystems fsck"
>```

### Build initramfs
Building initramfs requires knowing which kernel to use e.g. Linux 54. 

    # mkinicpio -p linux54

### Edit grub default
We could use the device naming but in systemd world this naming may not always be the same - *not guaranteed to be identical on every boot* - so it is highly recommended to use UUID.

To the UUID of the sda3 partition holding the cryptroot we use lsblk and define the output to be NAME,UUID.

    lsblk -o NAME,UUID /dev/sda3

You will get two UUIDs - the first being the physical partion - the second the cryptroot partition - and it is the UUID of the physical partition we need for grub.

>```
># nano /etc/default/grub
>```
>```
>GRUB_CMDLINE_LINUX="cryptdevice=UUID=xxxx-yyy-zzzz:cryptroot"
>```

### Install bootloader

    # grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Manjaro

### Generate grub config
    grub-mkconfig -o /boot/grub/grub.cfg

## Conclusion
---
Exit the chroot, umount device, close your encryption container

    # exit
    # umount -R /mnt
    # cryptsetup close cryptroot
    # reboot

Type in your passphrase for your encrypted container and login using root and the necessary password.

## Sources
---
* https://wiki.archlinux.org/index.php/Dm-crypt
* https://wiki.archlinux.org/index.php/Dm-crypt/Drive_preparation
* https://wiki.archlinux.org/index.php/Dm-crypt/Device_encryption
* https://www.howtoforge.com/tutorial/how-to-install-arch-linux-with-full-disk-encryption/
