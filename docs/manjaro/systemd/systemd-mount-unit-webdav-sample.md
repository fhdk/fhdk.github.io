---
title: 'WEBDAV mount unit'
taxonomy:
    category:
        - docs
---

### WEBDAV
---
See the Arch wiki on [webdav][4]. Install `davfs2` using a custom AUR buildscript

Edit the file `/etc/davfs2/secrets` and append your webdav service and credentials
```text
http(s)://address:<port>/path    davusername    "davpassword"
```
Or create a user file at `~/.davfs2/secrets` for user specific mounts

#### mount unit
Name the file according to `$YOUR_MOUNT_PATH.mount`
```text
[Unit]
Description=Mount WebDAV Service on server https://host:port/path
Wants=network-online.service

[Mount]
What=http(s)://address:<port>/path
Where=$YOUR_MOUNT_PATH
Options=rw,_netdev
Type=davfs
TimeoutSec=15

[Install]
WantedBy=remote-fs.target
WantedBy=multi-user.target
```

#### automount unit
Name the file according to `$YOUR_MOUNT_PATH.automount`
```text
[Unit]
Description=Mount WebDAV Service

[Automount]
Where=$YOUR_MOUNT_PATH
TimeoutIdleSec=300

[Install]
WantedBy=multi-user.target
```
[4]: https://wiki.archlinux.org/index.php/Davfs2