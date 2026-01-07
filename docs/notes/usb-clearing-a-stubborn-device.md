---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Clearing a stubborn storage device'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Possible methods to clear a disk

**WARNING:** 

To avoid someone copy pasting thus wiping their primary disk - replace $DEViCE with the actual device - don't append any partitions - just the device path. To get device paths

```bash
lsblk -no PATH
```

Execute as root - adding a failsafe layer to the listed commands - any one command should suffice.

1. **OPTION**

   Removes all traces of partition information including super blocks etc. 
   ```bash
   # sgdisk --zapall $DEVICE
   ```

2. **OPTION** 

   Write zeroes to the first 10M - effectively removing partition information 
   ```bash
   # dd if=/dev/zero of=$DEVICE bs=1M count=10
   ```

3. **OPTION**

   Creates a new GPT partition table with a single unformatted partition of Linux filesystem type
   ```bash
   # printf 'o\ny\nn\n\n\n\n\nw\ny\n' | gdisk $DEVICE
   ```

4. **OPTION**

   SSD memory cell cleaing
   * [https://wiki.archlinux.org/index.php/Solid_state_drive/Memory_cell_clearing][1]

[1]: https://wiki.archlinux.org/index.php/Solid_state_drive/Memory_cell_clearing

