---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'systemd-boot - BASIC - installation'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Yet another CLI guide
But what if you requirements are simple? And you want the installation to be as simple as possible? Plain and simple no fuzz - boot Manjaro - that's it.

**systemd-boot** is a bootloader which do not get much attention on Manjaro - since most Manjaro installations is created using Calamares installer which in turn installs grub. I recall a setting for an iso-profile setting the `efi_bootloader="grub"` - but it didn't work very well so I decided to learn how to implement systemd-boot the most simple way - later create a merge request to the tools.

## Before you begin
**First** - I am assuming you know your device path - for the safety of less experienced readers - I am using a device path **/dev/sdy**  you most likely do not find on your system.

**Second** - I am assuming you are using a root TTY as no commands is prefixed with **sudo**.

**TIP**: Don't use a graphical environment - switch to TTY - because the live system may lock screen and other unpleasant thing while you are using the terminal - thus breaking what ever you were doing.

**Third** - I will be using command line partitioning - no menu interfaces - pure command line.

**Fourth** - This guide will work for any device - it be internal, removable USB or otherwise attached to your system. To ease the pain of writing the same device over and over I made use of an environment variable - I assume you set the same too.

**TIP**
If your circumstances allows for it - you can use [ssh] to install remotely using another device on your network.


## Let's begin
If you have not done so already open a root TTY and set the device variable - remember it only exist in the current shell

    # INS="/dev/sdy"

Ensure your device is not mounted anywhere

    # umount -f "$INS"

## Now to the serious stuff
The stuff that needs disclaimers - you are on your own kind of stuff.

### Clean the disk's partition tables

    # sgdisk --zap-all "$INS"

### Create a new GPT partition table

    # sgdisk --mbrtogpt "$INS"

### Create the $esp partition

    # sgdisk --new 1::+512M --typecode 1:ef00 --change-name 1:"EFI System" "$INS"

### Create the root partition

    # sgdisk --new 2::: --typecode 2:8304 --change-name 2:"Linux x86-64 root" "$INS"

### Wipe everything from the partitions

    # wipefs -af "$INS"1
    # wipefs -af "$INS"2

### Format the partitions
Format $esp partition using FAT32

    # mkfs.vfat -vF32 "$INS"1

Format the the root partition using your preferred filesystem- If your device is flash based you can use f2fs which is created for flash or you can use ext4 which is a defacto standard for Linux.

    # mkfs.f2fs "$INS"2

### Mounting
Mount your root partition on the systems temporary mountpoint

    # mount "$INS"2 /mnt

Then create the folder for booting systemd ($esp)

    # mkdir /mnt/boot

And mount the $esp partition

    # mount "$INS"1 /mnt/boot

### Installing a base Manjaro system
This article is only scratching the surface of the new system. We only install a basic bootable system using the **base** meta package, filesystem tools for f2fs along with kernel and some required tools - and don't forget network connectivity

    # basestrap /mnt base f2fs-tools linux55 nano mkinitcpio bash-completion networkmanager systemd-boot-manager

## Configuring the base system
Configuring the system is the tedious - extremely boring - but crucial part, usually abstracted by tools like Manjaro Architect.

[details="The boring configurational steps"]
### Chroot into the mountpoint

    # manjaro-chroot /mnt /bin/bash

### Configurations
The **vconsole.conf** file contains information about the type of keymap you are using - in this case a danish keymap - but it could **us** for a default US english keymap.

    # echo KEYMAP=dk > /etc/vconsole.conf

The **hostname** file contains the name of your computer on a network - this must be unique - you can of course select another name

    # echo manjaro > /etc/hostname

The **hosts** file contains information local to your system. The is *almost* empty - edit the file and append below IP addresses and the hostname from your hostname file

    # nano /etc/hosts

>```
>127.0.0.1 localhost
>127.0.1.1 manjaro.localdomain manjaro
>```

The ever important **system time** - the example is for Denmark but it could be **Europe/Paris** if you live in that area.

    # ln -sf /usr/share/zoneinfo/Europe/Copenhagen

Unix systems expects the hardware clock to run in UTC and the system then corrects the clock using the timezone information - this is a point where Windows and Linux disagree causing trouble for dual-booters - which we are not.

    # hwclock --systohc

Enable the **network** and **timesync** (don't use `--now` in chroot, it will fail)

    # systemctl enable NetworkManager systemd-timesyncd

Now we create a **locale configuration** - this configuration defines system messages and how time, date and other units are displayed.

    # nano /etc/locale.gen

Uncomment the locales you want to use - e.g. using English for messages and German for date and time uncomment both. In this example - again for Denmark.

>```
>en_DK.UTF-8 UTF-8
>```

To actually use preferences the necessary files needs generated - this is done using the `locale-gen` command

    # locale-gen

The **locale.conf** file contains a reference to the locale files just created. Please see the Arch Wiki page on [locales] for additional entries you can add.

    # echo LANG=en_DK.UTF-8 > /etc/locale.conf

And finally set the root password

    # passwd

[/details]

## Booting

### [systemd-boot] on Arch Wiki

This is the interesting part you have worked yourself down to.

### initramfs
Use the mkinicpio command to generate the initramfs - it will copy the files to the boot ($esp) partition.

    # mkinitcpio -P

### bootloader
Now install the systemd bootloader to the boot ($esp) partition

    # bootctl --path=/boot install

The  rest of the configuration can be done outside chroot - necessary to write a boot entry to your EFI firmware

    # exit

For the bootloader to actually load we need create a configuration file to specify the kernel, initrd.

To avoid typos - use `ls` to list the content of boot folder and pipe the output to the boot configuration

    # ls /mnt/boot/init* /mnt/boot/vmlinuz* > /mnt/boot/loader/entries/manjaro.conf

Now open the file using nano

    # nano /mnt/boot/loader/entries/manjaro.conf

Amend the file to look like this (the order of the lines are not important)
>```
>title   Manjaro
>linux   /vmlinuz-5.5-x86_64
>initrd  /initramfs-5.5-x86_64.img 
>```

This new configuration file is then added to the file **loader.conf** 

    # nano /mnt/boot/loader/loader.conf

>```
>default manjaro
>```

## Maintenance
This article does not take into account the amd/intel microcode and maintenance due to kernel upgrades or booting different kernels.

To learn more - read up on [systemd-boot] on the Arch Wiki.

> Just a few things worth noting.
>
> * With systemd-boot, we also need to handle microcode loading by hand in the entries
> * It is probably worth pointing out that these entries will need to be added/updated as new kernels are installed and removed
> * Lastly, systemd-boot-manager will handle both those things for you in an automated fashion.  It can automate the installation of systemd-boot, the creation and removal of entries, the addition of microcode updates and has options setting defaults automatically.  It has full support for luks/lvm/btrfs/zfs/etc.
>-- @dalto 

## Finally

Unmount your devices

    # umount -R /mnt

If you are installing to an USB device - sync device before removing it

    # sync

And reboot

    # reboot

## Conclusion
You have only scratched the surface and there is work to be done - installing xorg, applications, themes - what ever you fancy - it's really up to you how this adventure ends.

Have fun - I did.

[locales]: https://wiki.archlinux.org/index.php/Locale
[systemd-boot]: https://wiki.archlinux.org/index.php/Systemd-boot
