---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Setting up NFS using Manjaro'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Goal
---
Sharing data between 2 computer in both directions. We will apply the sharing of the ~/Music folder and we will ensure there is no duplicated data. 

This guide is an application of the [Arch wiki guide to NFS](https://wiki.archlinux.org/index.php/NFS)

## Preparation
---
On Manjaro the package providing NFS is `nfs-utils` and  is installed as part of the system. As the NFS service works using either IP address or hostnames it is a good idea to test if you can ping a computer using its hostname.

We will use the client/server topology so select a system with a lot of available storage as server - as it makes sense to designate the available storage to as a network share.

A server needs to be powered up and visible on the network **and** have a predictable address on the network.

In this guide we assume you are using Network Manager and you set a static IP for your network card. You can use WiFi but it is not as good as a wired connection.

For this guide we assume
- network: 192.168.1.0
- subnet: 255.255.255.0
- server IP: 192.168.1.20
- computername: server01

## The share
---
As we want to avoid duplicating data we will move the relevant folder from the home folder to a designated structure and symlink the data into the home folder.

### Create the data structure
On the server create the folder structure and make the share point writable by world - in this example the Music folder.
```bash
# mkdir -p /data/Music
# chmod ugo+rwx /data/Music
```
Move the content of the ~/Music folder to the new folder - remove the empty ~/Music folder and symlink it to the new location
```bash
$ mv ~/Music/* /data/Music/
$ rm -f ~/Music
$ ln -sf /data/Music ~/Music
```
Verify it is done right by listing the content of ~/Music
```bash
$ ls ~/Music
```
### Create export folders
To share the folders using NFS we need an export point and the `/srv` folder is entrypoint. Create a folder to designate this is the NFS service point.
```bash
# mkdir -p /srv/nfs/Music
```
### Server bind mount
To avoid sharing a location which could expose the system we create a bind mount in the file system table binding the data folder with the nfs export point
> /etc/fstab
```text
/data/Music /srv/nfs/Music none bind 0 0
```
### Mount without restart
Remount the mount points using mount
```bash
# mount -a
```
### Export the shares
Edit the file `/etc/exports` share the root of the NFS and the share itself and access control for the shares.

NFS works only with IP and/or hostname restrictions so it is possible to restrict further down using only IP addresses or hostnames. The example below adapted from the Arch wiki - tested and tried - it works.

Additionally you can export the same share multiple times thus limiting access to a specific share to e.g. two devices - if the device uses a dynamic IP you can specifiy hostname and get the same result as if it uses static IP. In the example below we have allowed all computers on the network to connect to the Music share

>/etc/exports
```text
/srv/nfs            192.168.1.0/24(rw,sync,crossmnt,fsid=0)
/srv/nfs/Music      192.168.1.0/24(rw,sync)
```
### Service
Enable and start the service.
```bash
# systemctl enable --now nfs-server.service
```

## Client
---
Enable and start the service
```bash
# systemctl enable --now nfs-client.target
```

## Mounting
---
### Manual
Using a manual or fstab mount
- you would create a similar folder structure on the client

If you choose systemd units - systemd will take care of the folder creation - and all you have to do is symlink the folder when it has been mounted for the first time

```text
fh@ts:~|⇒  tree -L 2 /data
/data
└── nfs
    └── web
```
Manual mount the folder from the server
```bash
$ sudo mount -t nfs server01:/Music ~/data/nfs/music
```
Verify the share is up by listing the content of the mount
```
$ ls /data/nfs/music
```
If you want the shared music folder to appear directly as Music on client you can symlink directly to the clients Music folder - just ensure it is empty.
```bash
ln -sf /data/nfs/music ~/Music
```
### Mount using **fstab**
Using fstab requires the folder structure to present like a manual mount

The line in fstab could look like this
```text
server01:/nfs/music   /data/nfs/music  nfs auto,x-systemd.automount,x-systemd.device-timeout=10,timeo=14,x-systemd.idle-timeout=1min 0 0
```

### Using systemd units
The folder structure is not required as systemd will create it on first mount.
The system units are required to be named as the mount path followed by **.mount**
```bash
# touch /etc/systemd/system/data-nfs-music.mount
```
```text
[Unit]
Description=Music
After=network.target

[Mount]
What=server01:/nfs/music
Where=/data/nfs/music
Type=nfs
Options=_netdev,auto

[Install]
WantedBy=multi-user.target
```
Enable the mount
```bash
# systemctl enable data-nfs-music.mount
```

Then the automount unit (same name rule)
```bash
# touch /etc/systemd/system/data-nfs-music.automount
```
```text
[Unit]
Description=Automount music share
ConditionPathExists=/data/nfs/music

[Automount]
Where=/data/nfs/music
TimeoutIdleSec=10

[Install]
WantedBy=multi-user.target
```
Before you start the automount unit - ensure the share is not mounted - or the automount will fail.

Enable and start the automount
```bash
# systemctl enable --now data-nfs-music.automount
```
Navigate to the folder `/data/nfs/music` and verify the it has the content from your nfs service.

## Conclusion
---
You have now shared your collection of music on your local network. Any change to the ~/Music folder will immediately be reflected on the network.

## Background on the chosen folder tree
---
On the *nix filesystem we have  a location named *`/mnt`*. This location is mostly used to mount filesystems to chroot into, make changes and exit. The folder is described as a place for temporary mounts [Filesystem Hierachy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

> |Path| Usage|
> | --- | --- | 
> |`/mnt` | Temporarily mounted filesystems. |

The standard also suggests a folder for shared data
> |Path | Usage|
> | --- | --- | 
> | `/srv` | Site-specific data served by this system, such as data and scripts for web servers, data offered by FTP servers ... |

## The SysAdmin recommendation
---
What follows here is my personal experience as a sysadmin and believe me - make it simple - easily memorable - you can thank me later.

### Your data

As a safeguard of your system, the actual data should be located in a separate structure. You will later be adding bind mounts for them.

The data structure can - as starting point - be anything but sticking to the recognizable - use a distinctive pattern and simple rules

* Partitions mount in folders right below */data/*
* Shares from other systems in service/folder structure e.g. */data/nfs/data*
* Move local data to designated folders and use local symlinks to users home e.g. move the *~/Music* to */data/local/Music* and symlink the folder back *ln -s /data/local/Music ~/Music*

### Example
Starting in **/data/** your structure could look like this
```
/data >>> tree -L 2
.
├── build
├── nfs
│   └── data
├── smb
│   └── data
├── local
│   ├── Music
│   └── Video
├── projects
└── virtualbox
```
Mount your partitions using fstab

### Your shares

For the data you plan to share, create a similar structure using **/srv/** as base.

```
/srv >>> tree -L 2
.
└── nfs
     ├── Music
     └── Video
```

Add bind mounts to fstab - as bind mounts adds a layer of security to your shares - as you should avoid sharing anything which eventually could be used to traverse the root of your system.

```
/data/local/Music /srv/nfs/Music none bind 0 0    
/data/local/Video /srv/nfs/Video none bind 0 0
```
To avoid duplication of data you could create a symlink to your home folder - or - in case of indexing utilities like **Albert** which do not follow symlinks you could create an additional bind mount for the user(s) on the system.

As example - replace username with the actual username for which you would like to make the folder indexable.
```
/data/local/Music /home/username/Music none bind 0 0
```

### Further reading
https://wiki.archlinux.org/index.php/NFS
https://wiki.archlinux.org/index.php/samba

---
## Windows NFS Client
---
Based on the comments below - there are options to use Windows as NFS client. The forum do not recommend one solition for another and **do not support** such client.

* Windows 10 PRO and Enterprise Editions supports NFS
* Windows subsystem for Linux
* https://github.com/DeCoRawr/NFSClient/

## Revisions
* 2020-03-21
   - Fix error in mv command
   - Added systemd units
* 2019-09 Initial guide
