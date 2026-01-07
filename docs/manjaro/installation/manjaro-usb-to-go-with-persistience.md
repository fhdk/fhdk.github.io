---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'USB LXDE with persistence'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

# Manjaro-To-Go LXDE with persistence
 
This guide is different than the usual guides I write - the purpose of this is to give any of you a serious tool in case of displacement due to war, evacuation due to natural disasters, riots, ban on religious practice etc. You can also use the stick at home for your occasional secure computing - you can have a normal computer in home - and when necessary you can boot the stick - do what you need to do, e.g. keeping it up-to-date - shut it down and hide it for what ever prying eyes - governments, gangs, rebels, thieves - may force their way into your home.

With escalating instability of the world around us - the escalating impact human actions has on our environment - the ever  increasing possibilities of having to evacuate - many of us have emergency kits - sleeping bags, food supplies and water - ready to go - we sometimes forget our most vital belongings - the documents that defines us, our origin, our marriage, our children, photos of our relatives, photos of our passports, electronic copies of birth- and marriage certificates, our real-estate documents, proof of ownership for various items we carry - these invaluable documents we don't want others to get their hands on. Many of us values the Bible over everything and would want to have a copy - even an electronic copy - with us.

We can't rely on having a computer with us if we need to evacuate but we can rely on - should the need rise - that we can get access to a computer. But we cannot trust others with the stick - they could just copy the documents off the stick - we cannot trust a computer we have not booted to be clean - no keylogger, malware - we cannot trust it to be able to decrypt our data. The Xorg set of display drivers works with recent hardware - but due to the fast development of graphics hardware and though I expect it to work - obviously I cannot make any guarantee.

So this is - in my opinion - the ultimate guide to have a Linux in your pocket - an encrypted Linux - for storage of your personal documents.

## What is this about
I will demonstrate how to create a portable encrypted system using an USB device and the most minimal graphical environment possible using Manjaro.

## CAUTION
You will be doing the following as **root** so in case of device names - do double check your devices.

**IMPORTANT**: **Never just unplug your device** - you will damage the filesystem. If plugged into another operating system use the system file manager's eject method or ensure device data has been sync'd using the `sync` command. Then use `umount` to safely remove the device.

**DISCLAIMER**: I take no responsibility if you wreck something because you are to quick on the <kbd>Enter</kbd> key.

## Let us begin

### Change user
Open a terminal and login as root.
```bash
$ su -l root
Password: 
```
### Locating your device
Through the rest of this article - I will be using a device path of **/dev/sdy** - replace with your actual device. You can verify which device you are using by removing all USB flash devices. Insert the device you want to use and list your devices. You can recognize the removable device by the number **1** in the **RM** column. 

### Prepare the device
We will be using an unencrypted boot partition, so we cannot hide the presence of a Linux system on the device and where it is. Before we do anything we will fill the device using a random pattern. The benefit is that encrypted data cannot be distinguished from the rest of the device.

Start by unmounting your device - using force if necessary.

```bash
umount -f /dev/sdy
```

Wipe the device (double check device path) using a random pattern. 

```bash
dd if=/dev/urandom of=/dev/sdy bs=4M status=progress
```

### Partitions
You can use stick of your choice - you have to adjust the partition sizes accordingly.

For this article I am using a 64G SanDisk Extreme. To be able to exchange unencrypted data without having to boot the USB we will create a partition of 16G. To maximize compatibility we use exFAT which will be readable by most systems.

The intention is to create a hybrid USB capable of booting from a BIOS system as well as an EFI system so we need a special BIOS partition as well.

#### Create GUID partition table
```bash
sgdisk --mbrtogpt /dev/sdy
```

#### Create the bios boot partition
```bash
sgdisk --new 1::+1M --typecode 1:ef02 --change-name 1:"BIOS boot partition" /dev/sdy
```
#### Create the EFI system partition
```bash
sgdisk --new 2::+50M --typecode 2:ef00 --change-name 2:"EFI System" /dev/sdy
```

#### Create the data partition
```bash
sgdisk --new 3::+16G --typecode 3:0700 --change-name 3:"Microsoft basic data" /dev/sdy
```

#### Create grub boot partition
```bash
sgdisk --new 4::+1G --typecode 4:8300 --change-name 4:"Linux filesystem" /dev/sdy
```

#### Create root partition
```bash
sgdisk --new 5::: --typecode 5:8300 --change-name 5:"Linux filesystem" /dev/sdy
```

#### Create hybrid MBR
[Arch Linux documentation][1]
```bash
sgdisk --hybrid 1:2:3 /dev/sdy
```

#### Boot flag for data partition
```bash
sgdisk --attributes 3:set:2 /dev/sdy
```

#### Clean BIOS partition
```bash
wipefs -af /dev/sdy1
```

#### Clean and format EFI partition
```bash
wipefs -af /dev/sdy2
```
**FAT32**
```bash
mkfs.vfat -vF32 /dev/sdy2
```

#### Clean and format data partition
```bash
wipefs -af /dev/sdy3
```
**exFAT**
```bash
mkfs.exfat /dev/sdy3
```

#### Clean and format grub boot partition
```bash
wipefs -af /dev/sdy4
```
**ext2**
```bash
mkfs.ext2 /dev/sdy4
```

#### Create LUKS container
The larger `--iter-time` argument used will create a stronger resistance against brute-force but takes longer to decrypt.

* Example 1
   ```bash
   cryptsetup --verbose --hash sha256 --iter-time 2000 --use-random luksFormat /dev/sdy5
   ```
* Example 2
   ```bash
   cryptsetup --verbose --hash sha512 --iter-time 5000 --use-random luksFormat /dev/sdy5
   ```

Confirm and enter passphrase twice and unlock the container (longer password - better encryption)
```bash
cryptsetup open --type luks /dev/sdy5 cryptroot
```

Create an ext4 file system in the container
```bash
mkfs.ext4 /dev/mapper/cryptroot
```

### Mounting
Mount root
```bash
mount /dev/mapper/cryptroot /mnt
```

Create the `/boot` folder
```bash
mkdir /mnt/boot
```

Mount the grub boot partition
```bash
mount /dev/sdy4 /mnt/boot
```

Create the `/boot/efi` folder
```bash
mkdir /mnt/boot/efi
```

And mount the EFI partition
```bash
mount /dev/sdy2 /mnt/boot/efi
```

Finally create a folder for the data partition
```bash
mkdir /mnt/data
```

And mount the data partition
```bash
mount /dev/sdy3 /mnt/data
```

## Base installation
Replace **$LINUX** with the kernel of your choice.

e.g. `linux58` - `linux-latest` or `linux-lts`

```bash
basestrap /mnt base sudo networkmanager $LINUX links nano vim grub mkinitcpio bash-completion broadcom-wl ipw2100-fw
```

## Configure system
### Create fstab
```bash
fstabgen -U /mnt >> /mnt/etc/fstab
```
Verify the generated fstab has the expected content - remove references to devices which is **not** your USB **/dev/sdy** (e.g. the host systems swap - is often added).

### Chroot
```bash
manjaro-chroot /mnt /bin/bash
```

#### Console keyboard
Example for Denmark
```bash
echo LANG=dk > /etc/vconsole.conf
```
#### Hostname
```bash
echo manjaro > /etc/hostname
```

#### Edit **/etc/hosts**
```bash
nano /etc/hosts
```
```text
127.0.0.1   localhost
::1     localhost
127.0.1.1   manjaro.localdomain   manjaro
```

#### Timezone
Example for Denmark
```bash
ln -sf /usr/share/zoneinfo/Europe/Copenhagen /etc/localtime
```
```bash
hwclock --systohc
```
#### Network Manager
Enable network connection
```bash
systemctl enable NetworkManager
```

#### Enable ntp client
```bash
systemctl enable systemd-timesyncd
```

#### Locale
Locale example for Danish locale

-   uncomment en_DK.UTF-8 and en_US.UTF-8

Save file and generate the message table

```bash
nano /etc/locale.gen
```
```bash
locale-gen
```

#### /etc/locale.conf
Locale.conf example for Denmark
```bash
echo LANG=en_DK.UTF-8 > /etc/locale.conf
```

#### Root password
This is important - if you don't set it you will not be able to login as root - so the other option is to create a user with admin before rebooting. Pick a good password and do not reuse your luks key
```bash
passwd
```

#### /etc/mkinitcpio.conf

Add **encrypt** and **block** - the order is important - then save the changes

>```bash
># nano /etc/mkinitcpio.conf
>```
>```text
>HOOKS="base udev encrypt block keyboard autodetect modconf filesystems fsck"
>```
>-- https://wiki.archlinux.org/index.php/Mkinitcpio#Common_hooks

#### Build initramfs
```bash
mkinitcpio -P
```

### Install grub
**EFI**
```bash
grub-install --target=x86_64-efi --boot-directory=/boot --efi-directory=/boot/efi --removable --recheck
```

**BIOS**
```bash
grub-install --force --target=i386-pc --recheck --boot-directory=/boot /dev/sdy
```

**Fallback**
```bash
grub-install --force --target=i386-pc --boot-directory=/boot --recheck /dev/sdy3
```

### Edit grub default

We could use the device naming but in systemd world this naming may not always be the same - _not guaranteed to be identical on every boot_ - so it is highly recommended to use UUID.

To the UUID of the sdy5 partition holding the cryptroot we use lsblk and define the output to be NAME,UUID.

```bash 
lsblk -o NAME,UUID /dev/sdy5
```

You will get two UUIDs - the first being the physical partion - the second the cryptroot partition - and it is the UUID of the physical partition we need for grub.

> ```
> # nano /etc/default/grub
>  ```
> ```
> GRUB_CMDLINE_LINUX="cryptdevice=UUID=xxxx-yyy-zzzz:cryptroot"
> ```

Save the file and create grub config
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### USB specific considerations
Because we are using USB we know repeating writes is not healthy in the long run.

Switch journal configuration to use RAM and ensure the journal is not filling up the RAM.

```bash
nano /etc/systemd/journald.conf
```
Modify to include this and save the file
```
Storage=volatile
SystemMaxUse=16M
```
Edit your fstab and edit the options to include the **noatime** option. This will prevent writing to the filesystem every time a file changes which can be a lot.

```
nano /etc/fstab
```
Append to the options list like this - and save the file
```
# <file system>   <mount point>  <type>  <options>  <dump>  <pass>
UUID=sample-uuid   /              ext4    defaults,noatime 0 1
```

### Exit your chroot and eject the stick

```bash
sync
```
```bash
exit
```
```bash
umount -R /mnt
```
```bash
cryptsetup close /dev/mapper/cryptroot
```
```bash
sync
```

## Moment of truth
---
Verify the stick is bootable on another system at hand. Login as root.

If you are using a cable verify you have  a network connection.
```
nmcli device show | grep  IP4
```
If you need to create a wireless connection launch the Network Manager console
```
nmtui
```
Test your internet connection
```
links manjaro.org
```

If you cannot make a network connection - various Broadcom and RALink based devices comes to mind - you need to mount the stick in a chroot and install the necessary drivers and then test it again.

## Installing minimal GUI
---
The best GUI for this use case is LXDE. It is based on Openbox window manager and is well known for it's stability.

### Xorg and drivers
```bash
sudo pacman -Syu xorg-server xorg-server-common xorg-xinit xf86-video-amdgpu xf86-video-ati xf86-video-intel xf86-video-nouveau xf86-video-vesa xf86-input-libinput xf86-input-evdev
```

### LXDE 
LXDE can be installed using the a meta package so for this writeup it is the `lxde` package also adding some packages to make our life easier.

```
sudo pacman -Syu lxde epdfview accountsservice gnome-keyring gnome-icon-theme gnome-icons-standard perl-file-mimeinfo xdg-user-dirs xdg-user-dirs-gtk xdg-utils
```

### Spicing LXDE

```
sudo pacman -Syu lxde-wallpapers manjaro-lxde-config manjaro-lxde-desktop-settings manjor-lxde-logout-banner matcha-gtk-theme manjaro-openbox-matcha papirus-icon-theme papirus-maia-icon-theme ttf-dejavu ttf-roboto xcursor-breeze
```

### Network utilities

```
sudo pacman -Syu netctl ifplugd iw wpa_supplicant dialog network-manager-applet networkmanager-openvpn
```

## User
---
The reasoning creating the user lastly is the theming packages. Those packages are installed to `/etc/skel ` and used as skeleton when creating new users.

Choose a username and replace **$USERNAME** below with the chosen username
```bash
useradd -mUG lp,network,power,sys,wheel $USERNAME
```
Allow members of **wheel** group to perform administrative tasks

Run *visudo*
```bash
visudo
```
Locate the line reading *# %wheel ALL=(ALL) ALL* and remove the **#** in the beginning of the line
```bash
%wheel ALL=(ALL) ALL
```
And press <kbd>Esc</kbd><kbd>Shift</kbd><kbd>z</kbd><kbd>z</kbd>

Logout from the root session
```bash
exit
```

## Start X
---
Login with the new username and launch X
```bash
startx
```

Remember to shut the system down - don't remove the stick while it is running :slight_smile: 


[1]: https://bit.ly/2z7HBrP
