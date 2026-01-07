---
title: 'SMB mount unit'
taxonomy:
    category:
        - docs
---

### SMB network share
---
The package `smbclient` is enough. The credentials can be stored in location readable only by root. Replace the **$VARIABLES** with the values for your system

NOTE: According to the archlinux wiki the uid and gid can cause I/O errors. 
> **Warning:** Using `uid` and/or `gid` as mount options may cause I/O errors, it is recommended to set/check correct [File permissions and attributes](https://wiki.archlinux.org/index.php/File_permissions_and_attributes) instead. - https://wiki.archlinux.org/index.php/Samba#Manual_mounting

More information on Samba can be found on the [archlinux wiki][3]

#### SMB credentials
Create a file `/etc/smb.cred` or in user's home with content
```
user=$SMBUSER
password=$SMBPASS
workgroup=$WORKGROUP
```
Make the file readonly to owner
```
sudo chmod 600 /etc/smb.cred
```
#### SMB version
If needed you add a version to the options string e.g. __vers=NT1__

#### mount unit
Name the file according to `$YOUR_MOUNT_PATH.mount` 
```text
[Unit][Unit]
Description=NAS SMB video share

[Mount]
What=//$YOUR_SERVER/$YOUR_SHARE
Where=$YOUR_MOUNT_PATH
Type=cifs
Options=credentials=/etc/smb.cred,_netdev,iocharset=utf8,rw,file_mode=0777,dir_mode=0777
TimeoutSec=30

[Install]
WantedBy=multi-user.target
```

#### automount unit
Name the file according to `$YOUR_MOUNT_PATH.automount` 
```text
[Unit]
Description=Automount video share using SMB

[Automount]
Where=$YOUR_MOUNT_PATH
TimeoutIdleSec=10

[Install]
WantedBy=multi-user.target
```

[3]: https://wiki.archlinux.org/index.php/samba