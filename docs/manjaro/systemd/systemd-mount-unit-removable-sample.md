---
title: 'USB device'
taxonomy:
    category:
        - docs
---

### USB device partition
---
**[Type]** and **[Options]** are optional for disk devices.
To reduce unnecessary writings to SSD devices the **noatime** option is recommended.

#### mount unit
Name the file according to `$YOUR_MOUNT_PATH.mount` 
```text
[Unit]
Description=Mount USB backup device partition

[Mount]
What=/dev/disk/by-uuid/$UUID
Where=$YOUR_MOUNT_PATH
Type=ext4
Options=rw,noatime

[Install]
WantedBy=multi-user.target
```

#### USB partition automount unit
Name the file according to `$YOUR_MOUNT_PATH.autommount` 
```text
[Unit]
Description=Automount USB backup partition

[Automount]
Where=$YOUR_MOUNT_PATH
TimeoutIdleSec=10

[Install]
WantedBy=multi-user.target
```
