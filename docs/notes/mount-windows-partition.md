---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Access my second harddrive'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Your problem
So you have installed Manjaro and you realize that your second disk is not writeable. You can access the data but you cannot change it.

The primary cause is permissions and those can be solved by this guide.

However the issue could also arise when you dualboot with Windows, and the partition you want to access is the Windows root partition.

## NTFS partition
If your disk is a Windows disk - it can be as simple as 

- Restart your system and load Windows.
- Disable features like Fastboot, hibernation and hybrid sleep.
- Run a disk check and fix any errors

## Create a mountpoint
### Locate the device and partition(s).

    $ lsblk
    sda      8:0    0 223,6G  0 disk 
    ├─sda1   8:1    0   512M  0 part 
    ├─sda2   8:2    0    60G  0 part 
    ├─sda3   8:3    0   130G  0 part 
    └─sda4   8:4    0  33,1G  0 part [SWAP]
    sdb      8:16   0   477G  0 disk 
    └─sdb1   8:17   0 419,1G

### Make a permanent mountpoint.
Create a folder for the purpose (change all references to **mydisk** as you would want a more descriptive name).

    sudo mkdir -p /data/mydisk

### Mount the partition
Replace the *yX* with the device and the partition number you found.

    $ sudo mount /dev/sdyX /data/mydisk

### Set owner
Recursive change owner on the folder structure. Manjaro uses a group with the same name as the user which you can verify with:

    $ groups
    sys lp wheel network video optical storage scanner power $USER
    $ sudo chown -R $USER:$USER /data/mydisk

### Creating a permanent mountpoint for a partition
https://archived.forum.manjaro.org/t/wiki-howto-permanent-mountpoint-for-partition/26187


------
[Folders are locked, can access to them but can't delete permanently](https://archived.forum.manjaro.org/t/folders-are-locked-can-access-to-them-but-cant-delete-permanently/25964)

[Unable to access my secondary interal ssd on my laptop](https://archived.forum.manjaro.org/t/unable-to-access-my-secondary-internal-ssd-on-my-laptop/26012)
