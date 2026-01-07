---
title: 'Troubleshooting missing firmware access'
taxonomy:
    category:
        - docs
---

## UEFI firmware implementations

Some systems has a nasty habit of losing access to the firmware after your Linux installation. A couple of examples

* [Acer Community: Laptop lost firmware access][1]
* [Acer Community: Dual boot laptop lost firmware access][2]
* [Manjaro Community: Brandnew Acer laptop lost firmware access][3]
* [Manjaro Community: Acer laptop lost firmware access][4]
* [Manjaro Community: Fujitsu Siemens laptop lost firmware access][5]
* [Manjaro Community: Suddenly cannot enter bios, Boot Menu (Dell Inspiron 3511)][6]

This document is a work-in-progress ...

## The cause
It is not clear what makes this happen - a lot is pointing to bad implementations of the UEFI specification - in some cases the firmware is hardcoded to load `$esp/EFI/boot/bootx64.efi` [[wiki1]]

Your system will usually be able to boot your installed Linux - only the firmware is not displaying correctly.

Usually you should be able to load the firmware on next reboot by using
```
$ systemctl reboot --firmware-setup
```

## The cure
The first attempt to cure would be to reinstall grub - for an EFI system it goes like this
```
ESP=$(lsblk -no MOUNTPOINT | grep -e 'efi')
sudo grub-install --trage=x86_64-efi --efi-directory=$ESP bootloader-id=Manjaro --recheck
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo mkinitcpio -P
```
Next step is to check with LVFS if your firmware can be updated using Linux

Use [https://fwupd.org][100] to get familiar with the Linux Vendor Firmware Service. You can also search the database to verify if your system is supported.

There is no one-size-fits-all answer - the following is a list compiled from Manjaro Forum and the first being from Arch Wiki.

Read the Arch Wiki on UEFI and GRUB [[wiki2]] [[wiki3]] [[wiki4]]

I am listing the possibilities from least to most intrusive action.

1. Use the most basic approach
   - Switch user to root `su -` - provide the root password when requisted
   - Copy the grub or systemd-boot efi stub to the $esp/EFI/BOOT/BOOTX64.EFI
   - Try booting to the firmware using the aforementioned command
2. Removing the newly installed efistub
   - Boot your Linux system
   - Remove the grub efi loader - usually in a subfolder named for your distribution - e.g. `$esp/EFI/Manjaro/grubx64.efi` - rename the file or move it to another location
   - Try booting to the firmware
3. Remove everything from the $esp mount point
   - This requires having some kind of boot media readily available to be able to do some system rescue.
   - This has proven to work on at least 2 occasions and requires you to have some experience using chroot to resque the system
4. Using GRUB console to boot a removable media
   - This requires having some kind of boot media readily available and more intimate knowledge of the GRUB shell
   - It also requires the availability of either a Windows installation media or a dedicated rescue media like Hirens BootCD




[1]: https://community.acer.com/en/discussion/538305/cant-access-uefi-after-installing-linux-dual-boot
[2]: https://community.acer.com/en/discussion/575500/dual-boot-problem-bios-wont-load-if-i-install-linux-in-dual-boot-helios-300
[3]: https://forum.manjaro.org/t/new-laptop-but-after-install-no-bios-uefi-usb-boot-possible-anymore/56520
[4]: https://forum.manjaro.org/t/installing-manjaro-and-popos-in-dual-boot-with-windows-in-acer-laptop-breaks-bios/51740
[5]: https://forum.manjaro.org/t/after-install-bios-is-not-accessible-anymore/52160
[6]: https://forum.manjaro.org/t/suddenly-cannot-enter-bios-boot-menu/144966/2

[wiki1]: https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface#Troubleshooting
[wiki2]: https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface
[wiki3]: https://wiki.archlinux.org/title/EFI_system_partition#Troubleshooting
[wiki4]: https://wiki.archlinux.org/title/GRUB

[100]:https://fwupd.org