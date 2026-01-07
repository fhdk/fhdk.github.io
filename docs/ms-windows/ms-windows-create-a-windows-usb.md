---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Create a Windows USB'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Creating a bootable USB from a Windows ISO
The reason for needing this could be e.g. reinstalling Windows but it could also be to update your system firmware - because the vendor only provided Windows binaries - you need a Windows system.

To update you system firmware you can use a Windows PE environment like Hiren's BootCD <kbd>[Hirens BootCD][4]</kbd>

If you want to reinstall Windows after your Manjaro adventure get a <kbd>[Windows ISO from Microsoft][winiso]</kbd>

Only the manual approach described below was viable - until af few years ago when the github user [slacka][5] forked the WinUSB project. Thanks to his work the Linux community have an app to do abstract the CLI work.

Very receently a new tool has become available - the [ventoy project][6] - which makes the task even easier.

## Topics covered
1. ventoy bootable USB
2. WoeUSB
3. Manual using CLI

## 1. ventoy
[date=2020-05-09 time=12:35:00 timezone="Europe/Copenhagen"]
The ventoy utility is a great tool for booting a Windows ISO without having to jump through the hoops in this guide.

Install the **ventoy** package from repo

    sudo pacman -S ventoy

Locate your USB stick

    lsblk

Replace sdy below with your device

    sudo ventoy -i /dev/sdy

### Using a file manager
Using your file manager and drag your Windows ISO onto you USB and wait - patience is the keyword - patience.

When the copy operation is done - **use the eject button** in your file manager - and wait - wait until the device disappears from your file manager.

:exclamation: If you don't wait - data corruption will occur - and you don't want that.

### Using terminal
Using the device name from above mount the first partition to a temporary mount point

    sudo mount /dev/sdy1 /mnt

Copy the ISO file to the USB - assuming the ISO is in your Downloads folder

    cp ~/Downloads/<windows.iso> /mnt && sync
    
When the command finishes you can unmount the device

    sudo umount /mnt

:exclamation: If you don't wait - data corruption will occur - and you don't want that.

## 2. WoeUSB
Use the package woeusb available from [AUR][2]

    pamac build woeusb

Tip from @codesardine in a [comment][1]

Before launching the app

- open a terminal
- and execute

```
sudo modprobe -nv loop
```

Then launch the app and get your ISO done.

TIP:

* The process is unpacking files from the ISO and copying them to the USB
* The latest Windows ISO files contains a file larger than 4G - so **choose NTFS** as file system.

## 3. Manual using CLI

Remove all removable devices (USB), open a terminal and list known disk devices

    lsblk -la

Insert your USB stick and list your devices one more time

    lsblk -la

Make a note of the extra device listed. If you only have one disk then it probably  will be `/dev/sdb`.

**Please do double check the device id**

In the terminal clear the disk of any partition info, using this command (replace **sdy** with device letter from above).

    sudo dd if=/dev/zero of=/dev/sdy bs=1M count=10 oflag=sync

Then use fdisk to create the filesystem needed for the Windows ISO (replace X with device letter from above).

    sudo fdisk /dev/sdy

The commands in fdisk is as follows

[date=2019-11-01 time=17:43:00 timezone="Europe/Copenhagen"]
The partitioning may need rework due to single file inside Windows ISO is larger than 4G.

1. <kbd>o</kbd> - create a new empty DOS partition table
2. <kbd>n</kbd> - add a new partition
3. <kbd>Enter</kbd> - accept default partition type primary
4. <kbd>Enter</kbd> - accept default partition number 1
5. <kbd>Enter</kbd> - accept default first sector 2048
6. <kbd>Enter</kbd> - accept default last sector
7. <kbd>t</kbd> - change partition type
8. <kbd>c</kbd> - select W95 FAT32 (LBA)
9. <kbd>a</kbd> - set bootable flag for partition 1
10. <kbd>w</kbd> - write changes to disk

Newer versions of Windows 10 ISO contains a file bigger than 4G. Format the device using **exfat** or **ntfs** (replace **sdy** with device letter from above) to overcome the size limitation of FAT32.

    sudo mkfs.ntfs /dev/sdX1

Create a folder to mount your ISO

    mkdir ~/winiso

Mount your ISO

    sudo mount -o loop /path/to/windows/iso/filename.iso ~/winiso

Create a folder to mount your USB

    mkdir ~/winusb

Mount the partition (replace **sdy** with device letter from above)

    sudo mount /dev/sdy1 ~/winusb

Copy all files from ISO to USB

    cd ~/winiso
    cp -r * ~/winusb

The copy operation is going to take a long time depending on your USB port speed and your USB device.

When the copy is done ensure all data is flushed to the device using the sync command

    sync

When all data is flushed to the device you will be returned to the prompt.
Next thing is to move out of the winiso folder

    cd

Then unmount  the devices

    sudo umount ~/winiso ~/winusb

Remove the folders

    rm -rf ~/winiso ~/winusb

You should now be able to boot to your Windows install media.

## Revision

* Added [ventoy][6]
* Newer Windows media includes a wim file larger than 4GB
  - format to exfat or ntfs to be able to use with WoeUSB
* Fixed recursive flag on cleanup command

[1]: https://archived.forum.manjaro.org/t/howto-use-manjaro-to-create-a-bootable-windows-usb/92780/6?u=linux-aarhus
[2]: https://aur.archlinux.org/packages/?O=0&K=woeusb
[4]: https://www.hirensbootcd.org/
[5]: https://github.com/slacka/WoeUSB
[6]: https://ventoy.net
[winiso]: https://www.microsoft.com/en-us/software-recovery
