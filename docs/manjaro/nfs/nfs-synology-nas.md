---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Connect to Synology NFS share'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Setting up NFS on Synology NAS

### Access control
NFS do not use access control but IP address restriction.

## DSM control panel

### File Services
- Open **File Services** &rarr; 
- **enable NFS** &rarr;
- **Enable NFSv4** (do not add domain)
- Click **Apply**

### Shared Folder
- Open **Shared Folder** &rarr; 
- **Edit your shared folder** &rarr;
- **NFS Permissions** &rarr;
- **Add a single client based on IP**

#### Permissions
[details="Sample Screeenshot"]
![image|557x484](upload://ny28HTmwbjwn8j5QeYerFLYYHMc.png)
[/details]

* ReadWrite
* Map all to admin
* async no
* no priv port denied
* crossmount denied
 
You can add as many clients you need to any shared folder. 

Synology do not recommend using subnets as this give **any** client connected to your network same permissions as yourself.

## On your Manjaro client

Enable and start services to make Manjaro act as a NFS client.

    $ sudo systemctl enable --now nfs-client.target
    $ sudo systemctl enable --now NetworkManager-wait-online.service

List the available mounts on your diskstation

    $ showmount -e diskstation

[details="Sample output"]
```
❯ showmount -e diskstation
Export list for diskstation:
/volume1/data  192.168.x.x,192.168.x.y
/volume1/web   192.168.x.x,192.168.x.y
/volume1/video 192.168.x.x
/volume1/photo 192.168.x.x
/volume1/music 192.168.x.x
```
[/details]

### Create folder structure for mounting

    $ sudo mkdir -p /data/nfs/video /data/nfs/music /data/nfs/photo

Set permissions on the nfs sub folder structure

    $ sudo chmod -R ugo+rwx /data/nfs 

### Mounting

> **Note:** Server name needs to be a valid hostname (not just IP address). Otherwise mounting of remote share will hang. - [Arch wiki](https://wiki.archlinux.org/index.php/NFS#Manual_mounting)

Manually

    $ sudo mount -t nfs diskstation:/volume1/video /data/nfs/video

If you want to auto mount shares on client add the share to `/etc/fstab`

    diskstation:/volume1/video   /data/nfs/video  nfs auto,x-systemd.automount,x-systemd.device-timeout=10,timeo=14,x-systemd.idle-timeout=1min 0 0

## Conclusion
You should have an idea on how you make your Manjaro connect to your NAS using NFS.
