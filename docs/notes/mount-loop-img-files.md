---
title: 'CLI mount .img files on Arch based system'
date: '05:55 28-07-2023'
taxonomy:
    category:
        - docs
---

Mounting an image file e.g. an ARM device image

Use the first available loop device
```
losetup -f disk.img
```
List block devices to see which one got added e.g. loop0
```
partprobe /dev/loop0
```
Mount the partitions - e.g. a PinePhone image
```
mount /dev/loop0p2 /mnt
mount /dev/loop0p1 /mnt/boot
```
When done unmount
```
umount -r /mnt
```