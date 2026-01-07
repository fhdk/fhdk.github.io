---
published: true
date: '01-10-2019 00:00'
publish_date: '01-10-2019 00:00'
title: 'Dual boot Manjaro and Windows'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

**Difficulty: ★★☆☆☆**
## Dual boot - Step by Step
---

* Target systems
* Firmware
   * Checklist
* Windows preparation
* Manjaro installation
* Revisions

## Target systems UEFI
---
Computers with preinstalled Windows (Windows 10) is computers using UEFI firmware. This guide is a generic **guide targeted at UEFI installations**.

However some of the guide does apply even if you are using a BIOS/MBR setup.

If you are using a BIOS/MBR (DOS) partition schema watch out for this  _:information_source:  **Skip if using BIOS/MBR**_

To ensure a successful dual-boot installation using Windows and Manjaro there are a few steps to be taken.

## Installation type

:exclamation:  **DO NOT** mix UEFI with MBR partition scheme.

Always check your Windows root filesystem using `fdisk` on the live ISO. If the output contains ___Disklabel type: dos___ :warning: ___do not install as EFI___.

Windows may be installed in different ways which can affect the system various ways. The first clue you get using the fdisk command from the live ISO.

Windows 7 only supported DOS MBR partitions schema even the system did support EFI. Manjaro supports both GPT and DOS partitioning and it is very easy to start the Manjaro installer in EFI mode on a system supporting it.

To ensure a successful dual-boot on Windows 7 systems you **must** disable EFI in the firmware.

[details="Example for Windows 7 Home Premium 64bit in a VBoxVM"]
The disklabel type is **dos** which tells us this is a MBR partition and thus the system is booting from BIOS.

```
[manjaro manjaro]# sudo fdisk -l
...
Disklabel type: dos
Disk identifier: 0x9ab5fdd2
...
Device     Boot  Start      End  Sectors  Size Id Type
/dev/sda1  *      2048   206847   204800  100M  7 HPFS/NTFS/exFAT
/dev/sda2       206848 67106815 66899968 31.9G  7 HPFS/NTFS/exFAT
```
[/details]


## Firmware
---

The firmware is a crucial part of your system as it controls aspects on how the Linux kernel will interact with your the hardware. Some system firmware is setup in such a way that a Linux system does not recognize disk devices.

### Firmware checklist for BIOS systems

* :white_small_square: Use latest available firmware
* :white_small_square: Disable EFI
* :white_small_square: Disable RAID option
* :white_small_square: Enable AHCI

### Firmware checklist for EFI Systems

* :white_small_square: __Disable CSM (Legacy) boot__
* :white_small_square: __Disable Secure Boot__
* :white_small_square: Enable AHCI
* :white_small_square: Use latest available firmware
* :white_small_square: Disable **Optane Memory** and **Rapid Storage Technology** (RST)
* :white_small_square: Disable RAID option
* :white_small_square: Disable Fast Boot (if unable to boot from USB)

Some systems require the user to set a firmware password before more advanced options becomes available.

## Intel Optane Memory and Intel Rapid Storage Technology
Lately nvme devices has emerged labelled Intel Volume Management Device. These devices is a variation of Intel RST and requires the vmd module to be loaded for the OS to recognize it. As of  October 2021 Manjaro has added support for the vmd module when required. (https://gitlab.manjaro.org/packages/core/mkinitcpio/-/issues/1)

> **Is Linux* supported when using Intel® Optane™ memory for system acceleration?**
>
>No, the accelerated SATA drive must be running Windows 10 64-bit to use the Intel® Rapid Storage Technology (Intel® RST) driver software. This enables the supported/validated method of using the Intel® Optane™ memory for acceleration of the most commonly used data. Using the device with other software for caching is is not supported or validated. - https://www.intel.com/content/www/us/en/support/articles/000024018/memory-and-storage/intel-optane-memory.html

## BitLocker
:warning: BitLocker encryption is not compatible with a Manjaro Linux installation.

Without any prior knowledge or hands-on experience with BitLocker - we have enough evidence to discourage dual booting a BitLocker environment.

One possible reason a BitLocker environment won't boot could be that Secure Boot has to be disabled for the system to be able to boot Manjaro Linux.

If you depend on BitLocker - e.g. corporate requirement - don't install Manjaro as you will have to disable BitLocker to be able to use both systems.

## General Windows preparation
---

### Filesystem check
Linux is picky when it comes the Windows filesystem. Any inconsistencies in the filesystem and Linux will mount the filesystem **read-only**. The Windows command to fix the file system is

    chkdsk c: /F

### System clock
Configure your Windows installation to use UTC.
 
* https://archived.forum.manjaro.org/t/howto-get-your-time-timezone-right-using-manjaro-windows-dual-boot/89359

### Backup your documents

You can skip this but it is not recommended.
**Backup** any data you might want to keep **to an external location** of any kind.

### Partition cleanup

If you have experimented a lot and/or had a failed installation and/or you have a messy partition scheme you will have to manually delete those extra partitions with the Windows Disk Manager tool. Be careful that you do not delete partitions required by Windows or by an OEM recovery tool.

### Disk space
Use Windows disk tool to make room for a secondary Linux installation because Windows is the best tool to release space.

1. So boot into Windows.
2. Rightclick on **Start** &rarr; select **Disk Manager**
3. In Disk Manager - rightclick on your Windows drive **C:** &rarr; select **Shrink partition**
4. A reasonable size to release - depending on available space - would be 32768-65536 MiB (32-64GiB) or more.
5. When you are ready click **Shrink**

When you are done you are ready for the Manjaro installation. 

### Clean your Windows system

If you are like most users, your system came with Windows and your system has since been upgraded to Windows 10 (which leaves the old system behind). Major version upgrades - like 1804 - also leaves the old system behind and therefore a tremendous amount of dead data on your system that needs to be cleaned. 

1. Open Windows Explorer File manager and select **My Computer**.
2. Right click on you local drive **C:** &rarr; **Properties**
3. Click on **Disk Cleanup** button &rarr; wait
4. Click on the **Cleanup Systemfiles** &rarr; wait
5. **check all items** in the list (including the old Windows installation) &rarr; **OK**
6. Wait &rarr; wait until finished.
7. Close all windows

## Windows 10 preparation
---
### Disable Windows features
Do you plan on doing read/write on your Windows partition? Disable Windows options like

* Fast Startup
* Hybrid Sleep 

Windows **Hybrid Sleep** defaults to enabled on desktop computers and disabled for laptop computers.

**Why should I do that?** When Windows uses the above options it leaves the file system in a ***dirty state***. When the file system is in this state the Linux filesystem tool `ntfs-3g` mounts the file system **read-only**, effectively blocking you from making changes to your files on the Windows partition. To disable Windows Fast Startup you need to access the Windows Control Panel. You find it by clicking on **Windows Start** button &rarr; type **control** &rarr; select **Control Panel** desktop app.

In the Control Panel app

1. Click on **System and Security**
2. Click on **Power Options**
3. Click on **Choose what power buttons do**
  a. Click on **Change settings that are currently unavailable**
  b. Uncheck the option **Turn on fast startup**
4. Click on **Save Changes**

If for any reason you want to turn off hibernation completely
* Open command prompt as Administrator
* Input **powercfg /h off** and press **Enter**

## Installation considerations
---

Some of the choices presented here can be argued and the following two points I would like to address beforehand.

### Auto partitioning vs Manual partitioning

Some will argue that one should select the auto partition in the **Disk preparation** section of the installer.

>The strategy described here **ensures** no messing with the Windows EFI partition and therefore no problems with Windows removing the Manjaro boot loader.

### Separate root and /home
Separation of the system root and the home folder is not required but is another benefit of using manual partitioning. 

The separation of your personal data from the system - using a designated partition for the system's home folder makes it a bit easier to maintain your system. It is no secure replacement for a backup strategy it is just a handy solution should you decide to reinstall your system.

One pitfall here is making the root partition too small - using the recommended minimum size requires you to do regular system maintenance to avoid the system disk running full and thus making your system very hard to boot.

Depending on your available disk space your system root could be from 20-64GiB. The remaining is assigned to your personal data.

### Swap size

Setting a swap partition is the better choice because a little swap is - in most cases - better than none. 

The chosen size depends on your system, available RAM and disk type. Use the suggested size of 2 GiB ***or*** research and adjust accordingly to system, taste and need.

If you plan on using hibernation ensure the swap can hold system and graphics memory.

## Manjaro installation
---

Now that you have partition sizes defined let start and the numbers are MB which is the unit Calamares makes use of

1. Reboot your computer to the live USB media.
2. Launch the graphical installer - it is named Calamares.
3. Follow the guide until you reach the **Disk** selection/preparation
4. Select **Manual partitioning** &rarr; **Next**.
5. Select the correct disk selected - should be easy to see.
6. **EFI PARTITION**
    :information_source:  **Skip if using BIOS/MBR**
    Select the unpartitioned space  &rarr; **Create**
    a. **Size** &rarr; input **512**
    b. **Filesystem** &rarr; select **FAT32**
    c. **Mountpoint** &rarr; select **/boot/efi**
    d. **Flags** &rarr; check **boot** &rarr; **OK**
7. **SWAP PARTITION** 
    Select the unpartitioned space &rarr; **Create**
    a. **Size** &rarr; input **2048** 
    b. **Filesystem** &rarr; select **linuxswap** &rarr; **OK**
8. **ROOT PARTITION**
    Select the unpartitioned space &rarr; **Create**
    a. **Size** &rarr; input **20480** (min. recommended size)
    b. **Filesystem** &rarr; select **ext4**
    c. **Mountpoint** &rarr; select **/** (root) &rarr; **OK**
9. **HOME PARTITION**
    Select the unpartitioned space &rarr; **Create**
    a. **Size** &rarr; Use remaining
    b. **Filesystem** &rarr; select **ext4**
    c. **Mountpoint** &rarr; select **/home** &rarr; **OK**
 
:information_source:  **Skip if using BIOS/MBR**

* Continue with the guide and when finished **do not reboot**.
* Open a terminal
* Input `efibootmgr` &rarr; **Enter**
* Verify the **BootOrder** - you should have a **manjaro** entry and the corresponding number should be first in the **BootOrder**

## Before you reboot
---

:information_source:  **Skip if using BIOS/MBR**

* [Check your system's boot priorities](https://archived.forum.manjaro.org/t/wiki-windows-10-manjaro-dual-boot-step-by-step/52668/9?u=linux-aarhus)

### Oh No - It boots directly to Windows - What do I do?

:information_source:  **Skip if using BIOS/MBR**

Just boot to Windows. 

* Run CMD as Administrator
* `bcdedit /set {bootmgr} path \EFI\Manjaro\grubx64.efi`
* Reboot

## Bootloader repairs
---

If that not do the trick then @gohlip has a goldmine of tips to get grub bootloader right.

* https://archived.forum.manjaro.org/t/using-livecd-v17-0-1-as-grub-to-boot-os-with-broken-bootloader/24916

## Revisions
---
* Info on Intel RST - vmd module (2021-10-26)
* Discourage dual-boot BitLocker enabled (2020-11-20)
* Revision for Windows 7 (2020-11-10)
* Revision for Calamares 3.2.22.r7667
* Major revision (2020-03-13)
* Initial guide July 2018