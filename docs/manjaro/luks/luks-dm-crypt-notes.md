---
title: 'dm-crypt notes'
date: '08:54 14-01-2024'
taxonomy:
    category:
        - docs
---


## Error message
```
WARNING: keyslots area (998400 bytes) is very small, available LUKS2 keyslot count is very limited.
```
Check your partion - remove partition and increase the size.

## Error message
```
Cannot wipe header on device /dev/disk/by-partlabel/cryptsystem.
```
The device may be readonly. Execute as root
```
blockdev --getro <device>
```
Return 1 if read-only - 0 when read-write