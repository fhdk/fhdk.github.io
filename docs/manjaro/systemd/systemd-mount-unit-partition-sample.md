---
title: 'Partition mount unit'
taxonomy:
    category:
        - docs
---

### Disk PARTITION
---
**[Type]** and **[Options]** are optional for disk devices.
To reduce unnecessary writings to SSD devices the **noatime** option is recommended.

#### mount unit
Name the file according to `$YOUR_MOUNT_PATH.mount` 
```text
[Unit]
Description=Mount build partition

[Mount]
What=/dev/disk/by-uuid/$UUID
Where=$YOUR_MOUNT_PATH
Type=ext4
Options=rw,noatime

[Install]
WantedBy=multi-user.target
```