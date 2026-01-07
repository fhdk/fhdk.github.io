---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Minimal SMB configuration'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

### Minimal configuration for Manjaro Samba share

## Install samba package
Ensure the samba package is installed and the system is fully updated.

```text
sudo pacman -Syu samba
```

## Basic configuration
Create the config file **/etc/samba/smb.conf**
```text
sudo nano /etc/samba/smb.conf
```
Insert the following and save the file
```text
[global]
   workgroup = Manjaro
   server string = Samba Server
   server role = standalone server
   log file = /var/log/samba/log.%m
   max log size = 50
   guest account = nobody
   map to guest = Bad Password

[public]
   path = /srv/samba
   public = yes
   writable = yes
   printable = no
```
### Test your config
```text
sudo testparm /etc/samba/smb.conf
```
## Create share and set permissions
Create the shared folder
```text
sudo mkdir -p /srv/samba
```
Set permissions to any and all
```text
sudo chmod ugo+rwx /srv/samba -R
```
## Start the services
```text
sudo systemctl enable --now smb nmb
```
## Accessing the share
### Linux client
Ensure you have the *smbclient* package installed. Open your file manager and enter the servername or IP address in a filemanager's location bar using the smb protocol
```
smb://servername
```

### Windows client
Access the share using the filemanager and input the servername in this format - it is of course possible to use the IP address as well.
```
\\servername
```
## More reading
[Samba](https://wiki.archlinux.org/index.php/Samba#Server) on Arch Wiki

Sample Samba configuration with comments [smb.conf](https://git.samba.org/samba.git/?p=samba.git;a=blob_plain;f=examples/smb.conf.default;hb=HEAD) on samba.org
