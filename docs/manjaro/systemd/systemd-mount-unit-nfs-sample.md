---
title: 'NFS mount unit'
taxonomy:
    category:
        - docs
---

### NFS network share
---
#### mount unit
Name the file according to `$YOUR_MOUNT_PATH.mount` 
```text
[Unit]
Description=Mount NAS Video share using NFS

[Mount]
What=$YOUR_SERVER:/$YOUR_SHARE
Where=$YOUR_MOUNT_PATH
Type=nfs
Options=_netdev,auto

[Install]
WantedBy=multi-user.target
```

#### automount unit
Name the file according to `$YOUR_MOUNT_PATH.automount` 
```text
[Unit]
Description=Automount video share usuing NFS

[Automount]
Where=$YOUR_MOUNT_PATH
TimeoutIdleSec=10

[Install]
WantedBy=multi-user.target
```
