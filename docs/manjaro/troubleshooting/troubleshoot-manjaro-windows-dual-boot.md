---
title: 'Troubleshooting Manjaro Windows dual-boot'
taxonomy:
    category:
        - docs
---

## Symptom
After installation system boots directly to Windows.

Manjaro can be booted using the system's firmware boot-override.

## Possible cause

This could be due to Windows being installed using legacy boot on GPT.

When Manjaro is booted from USB on a EFI enabled system it will boot in EFI mode and install in EFI mode.

Most firmware which provides both Legacy boot (CSM or Compatibility Mode) and EFI will try Legacy before EFI and therefore Manjaro is only bootable when selected from the firmware boot override.

In theory - it should be possible to change system boot from EFI to Legacy, by manipulating the partition table.

## Possible solution
!! The following is untested - it is speculative but based on my knowledge of GPT partition table - no warrent you will succeed - manipulating the filesystem partition have the potential of being destructive - beware - there be dragons!

```bash
$ lsblk -lno NAME,PARTTYPENAME,MOUNTPOINT
```

To change from EFI to Legacy, by changing the EFI partition type from __0xEF00__ to __0x0700__ and run the grub installer for Legacy boot. This does not require booting from a live ISO - boot using your system's boot override to select Manjaro. Just in case the solution does not work - be sure to have a working bootable USB stick at hand to be able to revert the changes.

Login and open a terminal - first un-mount the EFI partition

```bash
# umount /boot/efi
```

Then edit your fstab and comment the line mounting your EFI partition

```bash
# nano /etc/fstab
```

Locate the line reading _UUID=x-y _ where _x-y_ is a placeholder the UUID on the system in question

    UUID=x-y                            /boot/efi          vfat    umask=0077 0 2

And add a __#__ in the beginning of the line

    #UUID=x-y                            /boot/efi          vfat    umask=0077 0 2

Press <kbd>F2</kbd> <kbd>y</kbd> to save the file

Then run a partition tool and change the partition type for the EFI partition. Depending on your system you need to replace $DEVICE with your actual device name

```bash
# cgdisk /dev/$DEVICE
```
First verify if a legacy boot partition exist - it is usually located at the beginning of the disk as type BIOS boot partition. The size is usually very small 1M sometimes 16M.

If it is not present you need to create it. While often located within the first MB's of the device this is not mandatory. What is mandatory is the partition type 0xEF02 and it must be unformatted.

If you have available 1024KiB space at the beginning of the disk you can create within that space

- select the empty space
- press Enter on  __`[ New ]`__
- press <kbd>Enter</kbd> twice
- enter the hex number __`EF02`__ and press Enter

Select the EFI partition

- press <kbd>Enter</kbd> __`[   Type   ]`__
- enter the hex number __`0700`__ and press <kbd>Enter</kbd>
- press <kbd>w</kbd> to write the change
- enter the word __`yes`__ in all lowercase and press <kbd>Enter</kbd>
- press <kbd>q</kbd> to exit

Run grub installer for legacy boot - target the primary device __`/dev/$DEVICE`__

```bash
# grub-install --target=i386-pc --recheck --boot-directory=/boot /dev/$DEVICE
```

Reboot the system
