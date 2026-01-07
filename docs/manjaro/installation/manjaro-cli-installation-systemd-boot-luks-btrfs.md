---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'systemd-boot - LUKS - btrfs'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Manjaro UEFI using systemd-boot, LUKS and btrfs

The following is my notes while reading and the changes I made to the subsequent installation to fit into a Manjaro system installation.

**NOTE**: Before you dive into btrfs - be sure to read the entire article - including the documentation linked at the end of this document.

## Overview
---
-  UEFI using systemd-boot
-  LUKS encrypted root
-  btrfs with subvolumes
-  Sources is listed at the end of this document

You can boot from any Manjaro ISO - open a terminal - and follow this guide.

For a similar guide without LUKS read here

* https://forum.manjaro.org/t/manjaro-uefi-systemd-boot-and-btrfs/116466?u=linux-aarhus

Assuming you know how to identify your disk devices and can replace the example device name **/dev/sdy** with the device for your system.

All commands written is assuming your are logged in as root. On Manjaro ISOs the root login is **root**:**manjaro**

## General
---
Connect to your network and ensure your system clock is correct
```
# systemctl start systemd-timesyncd
```
Set a preferred mirror and branch then download databases
```
# pacman-mirrors -aU https://manjaro.moson.eu -Sunstable
# pacman -Syy
```

## Partitioning and File System Creation
---
### Partition
Clear the disk of any existing file systems using random pattern - as the partition will be encrypted this will *disguise* the partitions and data held on it. It will take some time to complete - hours if you are having a big storage device.
```
# dd if=/dev/urandom of=/dev/sdy status=progress bs=10M
```
Use **cdisk** to create the boot partition and the main partition which will be encrypted
```
# cfdisk /dev/sdy
```
1. boot
    - 512M
    - EFI system partition type
 2. root
    -  remaining space
    - default type Linux file system

Set up the encryption container
```
# cryptsetup luksFormat /dev/sdy2
Are you sure? YES
Enter passphrase (twice)
# cryptsetup open /dev/sdy2 luks
```
### Format
Format to FAT32 for the boot and btrfs for the root
```
# mkfs.vfat -F32 /dev/sdy1
# mkfs.btrfs /dev/mapper/luks
```
The author of the original guide has some reasonable suggestions which also matches the defaults used by Manjaro Architect.
* **/** subvolume
* **/home** subvolume in case a root snapshot needs to be restored
* **/var** changes often so also a separate subvolume.
* **noatime** and **nodiratime** are used to prevent a write every time a file or directory is accessed (not great for a COW filesystem like btrfs).
* **zstd** is used for compression because it's fast and provides compression similar to xz.
* Don't use **discard**. Issue manual trim commands with fstrim or enable the fstrim.timer.
### Subvolumes
```
# mount /dev/mapper/luks /mnt
# btrfs subvolume create /mnt/@
# btrfs subvolume create /mnt/@home
# btrfs subvolume create /mnt/@var
# umount /mnt
# mount -o subvol=@,ssd,compress=zstd,noatime,nodiratime /dev/mapper/luks /mnt
# mkdir /mnt/{boot,home,var}
# mount -o subvol=@home,ssd,compress=zstd,noatime,nodiratime /dev/mapper/luks /mnt/home
# mount -o subvol=@var,ssd,compress=zstd,noatime,nodiratime /dev/mapper/luks /mnt/var
# mount /dev/sdy1 /mnt/boot
```
## Installation
---
### Install base Manjaro
```
# basestrap /mnt base btrfs-progs sudo manjaro-zsh-config intel-ucode networkmanager linux54 nano vim systemd-boot-manager mkinitcpio
```
#### Generate fstab
Generate fstab and verify the content
```
# fstabgen -U /mnt >> /mnt/etc/fstab
# cat /mnt/etc/fstab
```
## Configure system
---
### Chroot
```
# manjaro-chroot /mnt /bin/zsh
```
#### Hostname
```
# echo manjaro > /etc/hostname
```
#### Edit **/etc/hosts**
```
# nano /etc/hosts
```
```
127.0.0.1   localhost
::1     localhost
127.0.1.1   manjaro.localdomain   manjaro
```
#### Shell
```
# chsh -s /bin/zsh
```
#### Timezone
Example for Denmark
```
# ln -sf /usr/share/zoneinfo/Europe/Copenhagen /etc/localtime
# hwclock --systohc
```
#### Network Manager
Enable network connection
```
# systemctl enable NetworkManager
```
#### Enable ntp client
```
# systemctl enable systemd-timesyncd
```
#### Locale
Locale example for Danish locale
* uncomment en_DK.UTF-8 and en_US.UTF-8

Save file and generate the messages
```
# nano /etc/locale.gen
# locale-gen
```
#### /etc/locale.conf
Locale.conf example for Denmark
```
# echo LANG=en_DK.UTF-8 > /etc/locale.conf
```
#### Root password
```
# passwd
```
#### /etc/mkinitcpio.conf
Add **btrfs** and **encrypt** and save
```
# nano /etc/mkinitcpio.conf
```
```
HOOKS="base udev btrfs encrypt autodetect modconf block filesystems keyboard fsck"
```
```
# mkinitcpio -p linux54
```
#### systemd-boot
Install the boot loader
```
# bootctl --path=/boot install
```
#### Create Manjaro loaders
```
# sdboot-manage gen
```
Navigate to `/boot/loader/entries` and check the configurations **sdboot-manage** has created (there will be two).

:information_source: If you create the entries by hand please note
* **root=UUID=** is the UUID of your LUKS container
* **cryptdevice=UUID=** is the UUID of physical partition hosting your container

##### Default
```
title Manjaro Linux 5.4
linux /vmlinuz-5.4-x86_64
initrd /intel-ucode.img
initrd /initramfs-5.4-x86_64.img
options root=UUID=289ae676-7cbc-43a7-b4b7-e9cf325227c9 rw rootflags=subvol=/@ cryptdevice=UUID=9d336c58-0e8f-434d-b12a-b75663c4ad59
```
##### Fallback
```
title Manjaro Linux 5.4
linux /vmlinuz-5.4-x86_64
initrd /intel-ucode.img
initrd /initramfs-5.4-x86_64-fallback.img
options root=UUID=289ae676-7cbc-43a7-b4b7-e9cf325227c9 rw rootflags=subvol=/@ cryptdevice=UUID=9d336c58-0e8f-434d-b12a-b75663c4ad59:luks
```
### Base config done
```
# exit
# umount -R /mnt
# reboot
```

## Customizing
---
You should have a fully functioning Manjaro system. What comes next is your personal preferences. The example is a very basic vanilla Gnome desktop.

### Gnome desktop
```
# pacman -Syu xorg-server xorg-server-common xorg-xinit xorg-drivers accountsservice gnome-keyring gnome-session gnome-shell gnome-desktop gnome-terminal gdm
```
#### Add a user and set password
```
# useradd -mUG lp,network,power,sys,wheel -s /bin/zsh newuser
# passwd newuser
```
#### Admin user
Add user to wheel group
```
# visudo
```
##### Uncomment and save
```
%wheel ALL=(ALL) ALL
```
#### Enable displaymanager
```
# systemctl enable gdm
```
#### Reboot
```
# reboot
```

## Trouble shooting
---
If something went wrong and you need to get back in from the live image:
```
# cryptsetup open /dev/sdy2 luks
# mount -o subvol=@,ssd /dev/mapper/luks /mnt
# mount -o subvol=@home,ssd /dev/mapper/luks /mnt/home
# mount -o subvol=@var,ssd /dev/mapper/luks /mnt/var
# mount /dev/sdy1 /mnt/boot
# manjaro-chroot /mnt /bin/zsh
```
## drive preparation resource
* https://wiki.archlinux.org/index.php/Dm-crypt/Drive_preparation

## btrfs resources
---
* https://btrfs.wiki.kernel.org/index.php/Status
* https://btrfs.wiki.kernel.org/index.php/FAQ#Is_btrfs_stable.3F
* https://btrfs.wiki.kernel.org/index.php/Getting_started
* https://wiki.archlinux.org/index.php/Btrfs
* https://forum.manjaro.org/t/btrfs-tips-and-tricks/71186?u=linux-aarhus

## Credits
---
- https://austinmorlan.com/posts/arch_linux_install/

Credits in original source
- https://fogelholk.io/installing-arch-with-lvm-on-luks-and-btrfs/
- https://flypenguin.de/2018/01/20/arch-full-disk-encryption-btrfs-on-efi-systems/
- https://wiki.archlinux.org/
