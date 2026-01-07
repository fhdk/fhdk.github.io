---
title: 'FTP mount unit'
taxonomy:
    category:
        - docs
---

### MOUNT unit for FTP server
---
Using **[curlftpfs][1]**. It is also possible to store the username and password in a safe location readable only by root.

> You can put the user and password in a .netrc file in the home directory of the user that executes CurlFtpFS. It can have 600 permission. It's still clear text but at least is not accessible by all.
> The format is:
>
>     machine ftp.host.com
>     login myuser
>     password mypass

See [curlftpfs on archlinux wiki][3]. Replace the **$VARIABLES** with the actual values for your use case.

Name the file according to `$YOUR_MOUNT_PATH.mount`
```text
[Unit]
Description=Mount FTP server (ftp.server.tld) using curlftpfs
Wants=network-online.service

[Mount]
What=curlftpfs#ftp.server.tld
Where=$YOUR_MOUNT_PATH
Type=fuse
Options=rw,nosuid,uid=$UID,gid=$GID,allow_other,user=$FTPUSER:$FTPPASS

[Install]
WantedBy=remote-fs.target
WantedBy=multi-user.target
```

#### automount unit
Name the file according to `$YOUR_MOUNT_PATH.automount`
```text
[Unit]
Description=Automount ftp server ftp.server.tld

[Automount]
Where=$YOUR_MOUNT_PATH
TimeoutIdleSec=300

[Install]
WantedBy=multi-user.target
```
[1]: http://curlftpfs.sourceforge.net/
[2]: https://wiki.archlinux.org/index.php/CurlFtpFS