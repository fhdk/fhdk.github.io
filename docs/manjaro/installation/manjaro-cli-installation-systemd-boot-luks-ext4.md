---
published: true
date: '01-10-2019 00:00'
publish_date: '01-10-2019 00:00'
title: 'systemd-boot - LUKS - ext4'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
external_links:
    process: true
---

## Why another guide on encryption?
---
Using grub to boot an encrypted system often leads to long waits while grub decrypts the luks container just to get to the kernels.

This load time is a weakness of the current grub implementation - and while it will probably be solved in due time - we need to find ways around it.

For example you can use a separate partition for boot, $esp and root and leave boot unencrypted. This works and grub will happily boot. If you want to dual boot several variations of Linux and throw in a Windows and a couple of ISO - this is the way to go.

But what if you requirements are simple? You just want an encrypted Manjaro? And you want the installation to be as simple as possible?

**systemd-boot** is a bootloader which do not get much attention on Manjaro - since most Manjaro installations is created using Calamares installer which in turn installs grub. I recall a setting for an iso-profile setting the `efi_bootloader="grub"` - but it didn't work very well so I decided to learn how to implement systemd-boot the most simple way - later create a merge request to the tools.

## Before you begin
---
**First** - I am assuming you know your device path - for the safety of less experienced readers - I am using a device path **/dev/sdy**  you most likely do not find on your system.

**Second** - I am assuming you are using a root TTY as no commands is prefixed with **sudo**.

**TIP**: Don't use a graphical environment - switch to TTY - because the live system may lock screen and other unpleasant thing while you are using the terminal - thus breaking what ever you were doing.

**Third** - I will be using command line partitioning - no menu interfaces - pure command line.

**Fourth** - This guide will work for any device - it be internal, removable USB or otherwise attached to your system. To ease the pain of writing the same device over and over I made use of an environment variable - I assume you set the same too.

**TIP**
If your circumstances allows for it - you can use [ssh] to install remotely using another device on your network.

## Let's begin
---
If you have not done so already open a root TTY and set the device variable - remember it only exist in the current shell

    INS="/dev/sdy"

Ensure your device is not mounted anywhere

    umount -f "$INS"

### Now to the serious stuff
The stuff that needs disclaimers - you are on your own kind of stuff.

### Clean the disk's partition tables

    sgdisk --zap-all "$INS"

### Create a new GPT partition table

    sgdisk --mbrtogpt "$INS"

### Randomize the dev

    dd if=/dev/urandom of=$INS status=progress

### Create the $esp partition

    sgdisk --new 1::+512M --typecode 1:ef00 --change-name 1:"EFI System" "$INS"

### Create the root partition

    sgdisk --new 2::: --typecode 2:8304 --change-name 2:"Linux x86-64 root" "$INS"

### Wipe everything from the partitions

    wipefs -af "$INS"1

    wipefs -af "$INS"2

### Format the partitions
Format $esp partition using FAT32

    mkfs.vfat -vF32 "$INS"1

### Root LUKS container
Create the LUKS container - the `--iter-time` argument can be changed (default>10000) - the more iterations the better. `--tries` can be changed defaults to three (3) times.

    cryptsetup -v --iter-time 5000 --type luks2 --iter-time 50000 --hash sha512 --tries 5 --use-urandom luksFormat "$INS"2

Open the LUKS container

    cryptsetup open "$INS"2 cryptroot

Format the container using your preferred file system - If your device is flash based you can use f2fs which is created for flash or you can use ext4 which is a defacto standard for Linux.

    mkfs.ext4 /dev/mapper/cryptroot

### Mounting
Mount your LUKS container on the systems temporary mountpoint

    mount /dev/mapper/cryptroot /mnt

Then create the folder for booting systemd ($esp)

    mkdir /mnt/boot

And mount the $esp partition

    mount "$INS"1 /mnt/boot

### Installing a base Manjaro system
This article is only scratching the surface of the new system. We only install a basic bootable system using the **base** meta package, (if you use f2fs add the package `f2fs-tools`) along with kernel and some required tools - and don't forget network connectivity

    basestrap /mnt base linux510 nano mkinitcpio bash-completion networkmanager f2fs-tools

## Configuring the base system
---
Configuring the system is the tedious - extremely boring - but crucial part, usually abstracted by tools like Manjaro Architect.

### Chroot into the mountpoint

    manjaro-chroot /mnt /bin/bash

### Configurations
The **vconsole.conf** file contains information about the type of keymap you are using - in this case a danish keymap - but it could **us** for a default US english keymap.

    echo KEYMAP=dk > /etc/vconsole.conf

The **hostname** file contains the name of your computer on a network - this must be unique - you can of course select another name

    echo manjaro > /etc/hostname

The **hosts** file contains information local to your system. The is *almost* empty - edit the file and append below IP addresses and the hostname from your hostname file

    nano /etc/hosts

>```
>127.0.0.1 localhost
>127.0.1.1 manjaro.localdomain manjaro
>```

The ever important **system time** - the example is for Denmark but it could be **Europe/Paris** if you live in that area.

    ln -sf /usr/share/zoneinfo/Europe/Copenhagen /etc/localtime

Unix systems expects the hardware clock to run in UTC and the system then corrects the clock using the timezone information - this is a point where Windows and Linux disagree causing trouble for dual-booters - which we are not.

    hwclock --systohc

Enable the **network** and **timesync** (don't use `--now` in chroot, it will fail)

    systemctl enable NetworkManager systemd-timesyncd

Now we create a **locale configuration** - this configuration defines system messages and how time, date and other units are displayed.

    nano /etc/locale.gen

Uncomment the locales you want to use - e.g. using English for messages and German for date and time uncomment both. In this example - again for Denmark.

>```
>en_DK.UTF-8 UTF-8
>```

To actually use preferences the necessary files needs generated - this is done using the `locale-gen` command

    locale-gen

The **locale.conf** file contains a reference to the locale files just created. Please see the Arch Wiki page on [locales] for additional entries you can add.

    echo LANG=en_DK.UTF-8 > /etc/locale.conf

And finally set the root password - do not skip as you will not be able to login into the rebooted system.

    passwd

## systemd-boot
---
* [systemd-boot] on Arch Wiki

This is the interesting part you have worked yourself down to. Because we are using LUKS encrypted root partition we need to make some system parts available at boot time.

We need to make the **initramfs** aware of the encryption we use and it need to accept input from the user on encryption phrase used to decrypt the system.

### initramfs
Those settings is defined in the file **mkinitcpio.conf** -  we need to edit that file so suit our purpose

    nano /etc/mkinitcpio.conf

Edit the **HOOKS** line to include *keyboard*, *keymap*, *sd-vconsole* and *sd-encrypt*. This is required to get past the decryption phase. And the order of appearance is important - they must appear *before* autodetect.
>```
>HOOKS="systemd keyboard keymap sd-vconsole block sd-encrypt autodetect modconf filesystems fsck"
>```

Use the mkinicpio command to generate the initramfs - it will copy the files to the boot ($esp) partition.

    mkinitcpio -P -d /boot

### bootloader
Now install the systemd bootloader to the boot ($esp) partition

    bootctl --path=/boot install

To get the UUID for the device we need to exit chroot and the rest of the configuration can be done outside.

    exit

For the bootloader to actually load we need create a configuration file to specify the kernel, initrd and kernel options.

The configuration must match your system's kernel and initrd also DEVICE-UUID must the UUID of the physical device hosting the LUKS container. We can use `lsblk` to output the UUID and write it directly to the entry configuration.

    lsblk -no PATH,UUID "$INS"2 > /mnt/boot/loader/entries/manjaro.conf

We also need the filenames of the kernel and to avoid typos - we use `ls` and pipe the output to the same file - just appending instead

    ls /mnt/boot/init* /mnt/boot/vmlinuz* >> /mnt/boot/loader/entries/manjaro.conf

Now open the file using nano

    nano /mnt/boot/loader/entries/manjaro.conf

Amend the file to look like this (the order of the lines are not important)
>```
>title   Manjaro
>linux   /vmlinuz-5.10-x86_64
>initrd  /initramfs-5.10-x86_64.img 
>options root=/dev/mapper/cryptroot rd.luks.name=DEVICE-UUID=cryptroot
>```

This new configuration file is then added to the file **loader.conf** 

    nano /mnt/boot/loader/loader.conf

>```
>default manjaro
>```

## Maintenance
---
This article does not take into account the amd/intel microcode and maintenance due to kernel upgrades or booting different kernels.

To learn more - read up on [systemd-boot] on the Arch Wiki.

To facilitate tedious maintenance tasks you can install the **systemd-boot-manager** package from official repo.

> Just a few things worth noting.
>
> * With systemd-boot, we also need to handle microcode loading by hand in the entries
> * It is probably worth pointing out that these entries will need to be added/updated as new kernels are installed and removed
> * Lastly, systemd-boot-manager will handle both those things for you in an automated fashion.  It can automate the installation of systemd-boot, the creation and removal of entries, the addition of microcode updates and has options setting defaults automatically.  It has full support for luks/lvm/btrfs/zfs/etc.
>-- @dalto 

## Test your install
---
Unmount your devices

    # umount -R /mnt

Close the LUKS container

    # cryptsetup close /dev/mapper/cryptroot

If you are installing to an USB device - sync device before removing it

    # sync

And reboot

    # reboot

## Install Manjaro Edition
---
You now have a the minimal for a system to boot and make a network connection - where you go from here is really up to you.

### Option 1
Pick from the big-big box of Linux Lego and customize your own favorite system.

### Option 2
On your favorite system you can use two files from your root

* /rootfs-pkgs.txt
* /desktopfs-pkgs.txt

Sanitize the lists combining them into one list - then feed them to pacman to install them into your new root file system.

Start with root filesystem and pipe it into a new file
```
cat /rootfs-pkgs.txt | awk '{print $1;}' > ~/iso-pkglist.txt
```
Continue with the desktop filesystem and append it to the file
```
cat /desktopfs-pkgs.txt | awk '{print $1;}' >> ~/iso-pkglist.txt
```
Then feed the entire list to pacman and direct the installation to your mounted root using the convenient options of not confirming and only install if not already installed.
```
pacman -Syy --no-confirm --needed --root /mnt - < ~/iso-pkglist.txt
```
You will now require a few extra steps - which requires chroot. 

### Administrative user
It is important to hold back on user creation until you have installed the packages making up the Manjaro look and feel otherwise your user will have only desktop defaults. All theming etc. is stored in `/etc/skel` and upon user creation these files as used as a skeleton.

The first created should be an administrative user - replace $USERNAME with an actual username
```
useradd --create-home --user-group -G network,scanner,lp,wheel $USERNAME
```

Then set a password for the user
```
passwd $USERNAME
```
Display manager

To enable a graphical login you must enable the display manager. Which one depends on the environment

#### KDE
```
systemctl enable sddm
```
#### Xfce
```
systemctl enable lightdm
```
#### Gnome
```
systemctl enable gdm
```

## Conclusion
---
You have only scratched the surface and there is work to be done - installing Xorg, applications, themes - what ever you fancy - it's really up to you how this adventure ends.

Have fun - I did.

[locales]: https://wiki.archlinux.org/index.php/Locale
[systemd-boot]: https://wiki.archlinux.org/index.php/Systemd-boot
[ssh]: https://forum.manjaro.org/t/howto-luks-encrypted-system-using-systemd-boot/124703/2?u=linux-aarhus
