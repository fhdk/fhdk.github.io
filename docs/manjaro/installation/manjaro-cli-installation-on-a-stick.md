---
title: 'Manjaro on a stick'
taxonomy:
    category:
        - docs
published: true
date: '2019-11-01 19:08'
publish_date: '2020-09-01 19:10'
---

## Run Manjaro on a stick

This is my learning experience - documented for those readers who like the adventure :slight_smile:
 
This guide and the resulting USB is intended to run on generic hardware - any **EFI based computer** you may be close to and capable of booting an USB by using the firmware's boot override. You can of course extend this crash guide in any way you want - please - that is why I created it - to be the base for experiments - for learning - for the street credit you gain. 

**Please have fun!**

## Headlines
1. Goals
2. First steps
3. Locate device
4. Preparation
5. Partitioning
6. Formatting
7. Mounting
8. Base installation
9. Configure for USB
10. Moment of truth
11. Install LXDE
12. Add ISO image to Grub
13. Package list
14. Sources

## 1. Goals
---
1. Minimal Manjaro on USB stick - verify boot (**EFI**)
2. Data sharing with other systems using exfat
3. Extend Manjaro with Lxde using CLI login and startx
4. Extend Grub to boot ISO files (Manjaro - Tails)

> [[HowTo] Install Manjaro using CLI][1]

I have used a Kingston USB3 device 32G and my test system is an Yepo with Intel J3455, 8GB RAM and USB2 and [USB3][2]. 

Due to how USB sticks are designed - it may be more feasible to load an ISO and use Manjaro in its unaltered live environment - but let's find out.

## 2. First steps
---
### CAUTION
You will be doing the following as **root** so in case of device names - do double check your devices.

**DISCLAIMER**: I take no responsibility if you wreck something because you are to quick on the <kbd>Enter</kbd> key.

### Change user
Open a terminal and login as root.
```bash
$ su -l root
Password: 
```
I am using a device path **/dev/sdl** through the rest of the guide - replace with your actual device. You can verify which device you are using by removing all USB flash devices. Insert the device you want to use and list your devices. You can recognize the removable device by the number **1** in the **RM** column. 

## 3. Locate the device
---
It is very important you verify your device path. You will be so sorry if accidentally zap the wrong device.

### Sample device list
```bash
# lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
...
sdc      8:32   0 447,1G  0 disk 
├─sdc1   8:33   0   300M  0 part /boot/efi
├─sdc2   8:34   0 412,4G  0 part /
└─sdc3   8:35   0  34,4G  0 part [SWAP]
...
sdl      8:176  1  28,8G  0 disk 
├─sdl1   8:177  1     8G  0 part 
├─sdl2   8:178  1    20G  0 part 
└─sdl3   8:179  1 837,5M  0 part 
...
```

## 4. Preparation
---
Wipe the device (double check device path) (for encrypted system use urandom instead of zero's)
```bash
# dd if=/dev/zero of=/dev/sdl bs=4M status=progress
```

## 5. Partitioning
---
**IMPORTANT**:
* The partition types **must** be set.
* Partitions **must** be created in the following order.
  (Microsoft expects only one partition on USB - the first)

Run *cfdisk* and select **gpt** when prompted.
```bash
# cfdisk --zero /dev/sdl
```
* First (1) partition - 8G
  * Partition type: **Microsoft basic data**
    * Filesystem **exfat**
* Second (2) partition - 20G
   * Partition type: **Linux root (x86-64)**
      * Filesystem **f2fs**
 * Third (3) partition - remaining space
    * Partition type: **EFI System**
      * Filesystem **fat32**

## 6. Formatting 
---
### Partition 1 
```bash
# mkfs.exfat /dev/sdl1
```
### Partition 2 
```bash
# mkfs.f2fs /dev/sdl2
```
### Partition 3
```bash
# mkfs.fat -F32 /dev/sdl3
```
## 7.  Mounting
Mount root
```bash
# mount /dev/sdl2 /mnt
```
Create the `/boot/efi` folder
```bash
# mkdir -p /mnt/boot/efi
```
And mount the EFI partition
```
# mount /dev/sdl3 /mnt/boot/efi
```

## 8. Base installation
---
Use **section 6 & 7 in the [CLI guide][1]** to install the base system and return here without exiting the chroot.

**IMPORTANT**: Install `f2fs-tools` in your root.

(The complete list of package can be found at the end of the guide.)

## 9. Configure for USB
---
### Edit *mkinitcpio.conf*
>```bash
># nano /etc/mkinitcpio.conf
>```
>```bash
>MODULES=(f2fs)
>...
>HOOKS="base udev block keyboard autodetect modconf keymap filesystems fsck"
>```
>-- Source: Arch Wiki [Mkinitcpio Common hooks][10]

### Build initramfs
```bash
# mkinitcpio -p linux53
```
### Install grub
**64-bit EFI**
```bash
# grub-install --target=x86_64-efi --removable --recheck --boot-directory=/boot --efi-directory=/boot/efi
```
**32-bit EFI**
```bash
# grub-install --target=i386-efi --removable --recheck --boot-directory=/boot --efi-directory=/boot/efi
```
>-- Source: Arch Wiki [GRUB Installation part 2][9]

### Create grub config
Because we are using **f2fs** we need to modify the grub defaults. Later on we also want to show the grub menu for selecting an ISO to load and we do not want that grub menu to disappear too fast.
>```bash
># nano /etc/default/grub
>```
>```bash
> GRUB_DEFAULT=0
> GRUB_TIMEOUT=-1
> GRUB_TIMEOUT_STYLE=menu
> ...
> GRUB_CMDLINE_LINUX_DEFAULT=""
> ...
># GRUB_SAVEDEFAULT=true
>```

Save the file and create grub config
```bash
# grub-mkconfig -o /boot/grub/grub.cfg
```

## 10. Moment of truth
---
Close the chroot
```bash
# exit
```

### Moment of truth

Verify the stick is bootable on another system at hand. Login as root.

If you are using a cable verify you have  a network connection.
```
nmcli device show | grep  IP4
```
If you need to create a wireless connection launch the Network Manager console
```
$ nmtui
```
If you followed section 6 in the CLI guide to the letter you have a text browser. Use it to test your internet connection
```
$ links manjaro.org
```

If you cannot make a network connection - wireless Broadcom comes to mind - you need to mount the stick in a chroot and install the necessary drivers and then test it again.

When you have verified the stick works you are ready to continue.

## 11. Extend with LXDE
---
Next thing is to extend our system from a basic CLI to a basic GUI using only a basic set of packages and a GUI web browser. I have chosen Lxde because it is lightweight, easy to use and Midori because of the small footprint.

* [[HowTo] Install a basic LXDE][4]

When all packages has been installed
* in chroot 
  * exit chroot, unmount the partitions and boot a testing device
  * **Never just unplug your device** - you will damage the filesystem.
  * Wait until umount has finished and for eventualities execute the **sync** command.

### Reboot
Restart your system and boot to the stick.

Login with the new username and launch X
```bash
$ startx
```
You are now running **Manjaro-on-a-stick** and we can have some more fun.

## 12. Extend grub to boot ISO images
---
I am assuming the following

1. You have booted the USB stick - it is working.
2. You have a running X.
3. You have a working internet connection.
4. You know how to override your boot sequence.
5. You know how to access the grub menu using the <kbd>Shift</kbd> key.

Most of following be done from the booted USB or using chroot. You may prefer to use a mounted environment to do the editing and copying the ISO to the USB as doing it using the USB's environment maybe to slow for you.

I have used a root shell and a temporary mount - but you can do as you see fit. If you choose to work directly on the booted USB you will need additional packages installed and adapt the paths used below.

### Get Tails
Grub can boot ISO image - right off the stick. Let's see if can get Tails to boot.

Open Midori and download the latest Tails ISO from the [official  website][8]. You find link for the **ISO image** near the bottom of the page. The image is the same for DVD and virtual machines.
**TIP**: Midori shows no sign of beginning the download. Don't click download but check in the upper right corner of the browser - the download symbol - it should show the progressing download.

**IMPORTANT**: NEVER download Tails from other sources than the official.

### The ISO folder
When you have downloaded the ISO move the file to a new folder you are about to create in the `/boot` folder. Execute the following command in the LX terminal emulator.
```
sudo mkdir /mnt/boot/isos
```
Move the file to the new location - we use a wildcard because this guide cannot assume a specific version.
```
sudo mv ~/Downloads/tails*.iso /mnt/boot/isos
```
Verify you actually moved it.
```
ls /mnt/boot/isos
```
You can copy any number of ISO files to your USB - your limit is only the amount of space allocated to your root environment.

### The grub configuration
This part makes use of a set of scripts called **multibootusb**. They are available from github user @aguslr (see reference #2) and was brought to my attention some time ago by @AgentS (see reference #3).

Clone the the multiboot usb repo using git
```
git clone https://github.com/aguslr/multibootusb
```
Copy the configuration files from the multibootusb repo to your USB mount.
```
sudo cp -r mbusb.{cfg,d} /mnt/boot/grub
```
**TIP**: The `mbusb.cfg` can take some time to load. Only use scripts for ISOs you plan to use. In this case I only copied the folders `manjaro.d` and `tails.d` - you are free to copy others if you like.

I have not tested all possibilities - so I don't know if it all works. Please direct multibootiso script issues to the author.

Create a file on the USB named *custom.cfg*

>```
># nano /mnt/boot/grub/custom.cfg
>```
>```
># ========= Multiboot ISO  ==========
># https://github.com/aguslr/multibootusb
>
>if [ -e "$prefix/mbusb.cfg" ]; then
>  source "$prefix/mbusb.cfg"
>fi
>```

* Save the file and close it.
* Then unmount your partition
* Eject the stick
* Boot your test system

### Booting Manjaro ISO
The boot options for Manjaro works

### Booting Tails ISO
The boot options for Tails does not work on my test system - it seems those change from time to time or more likely I am doing something wrong. I have figure out how to make this work.

## 13. Package list
[details="Complete package list"]
>base linux53 dhcpcd networkmanager grub efibootmgr f2fs-tools mkinitcpio vi nano sudo links xorg-server xorg-server-common xorg-xinit xorg-drivers ipw2100-fw ipw2200-fw iw iwd accountsservice engrampa epdfview galculator gnome-keyring gnome-icon-theme gnome-themes-standard gpicview-gtk3 leafpad lxappearance-gtk3 lxde-common lxde-icon-theme lxhotkey-gtk3 lxinput-gtk3 lxlauncher-gtk3 lxmed lxmenu-data lxpanel-gtk3 lxsession-gtk3 lxtask-gtk3 lxterminal manjaro-icons obconf pcmanfm-gtk3 perl-file-mimeinfo xdg-user-dirs xdg-user-dirs-gtk xdg-utils firewalld midori network-manager-applet lxde-wallpapers manjaro-lxde-config manjaro-lxde-desktop-settings manjaro-lxde-logout-banner matcha-gtk-theme manjaro-openbox-matcha papirus-icon-theme papirus-maia-icon-theme ttf-dejavu ttf-roboto xcursor-breeze
[/details]

## Sources
* [Install Manjaro using CLI][1]
* [Install Manjaro using LUKS encryption][3]
* [Install a basic LXDE][4]
* [Manjaro Lxde iso-profile][5] 
* [Grub multiboot script][6]
* [Makeshift restore partition][7]
* [Tails official web][8]
* [Arch Wiki - GRUB][9]
* [Arch Wiki - mkinitcpio][10]
* [Install a basic Gnome][11]

[1]: https://archived.forum.manjaro.org/t/howto-install-manjaro-using-cli-only/108203?u=linux-aarhus
[2]: http://www.wyae.de/docs/boot-usb3/]
[3]: https://archived.forum.manjaro.org/t/howto-install-encrypted-manjaro-using-cli/110553?u=linux-aarhus
[4]: https://archived.forum.manjaro.org/t/howto-install-a-basic-lxde-gui/110139?u=linux-aarhus
[5]: https://gitlab.manjaro.org/profiles-and-settings/iso-profiles/tree/master/community/lxde
[6]: https://github.com/aguslr/multibootusb
[7]: https://archived.forum.manjaro.org/t/create-a-makeshift-restore-and-iso-booting-partition-with-grub/65370?u=linux-aarhus
[8]: https://tails.boum.org/install/index.en.html
[9]: https://wiki.archlinux.org/index.php/GRUB#Installation_2
[10]: https://wiki.archlinux.org/index.php/Mkinitcpio#Common_hooks
[11]: https://archived.forum.manjaro.org/t/howto-install-vanilla-gnome/116548?u=linux-aarhus