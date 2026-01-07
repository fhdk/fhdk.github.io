---
published: true
date: '01-10-2019 00:00'
publish_date: '01-10-2019 00:00'
title: 'systemd - mount units'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Disk device recognition
Arch uses systemd and udev (see [Arch Wiki][1]) to load devices at boot time. The loading order of devices is arbitrary and therefore you cannot predict which device will be available at a given path.

But as explained on the Arch wiki [static device names][2] do exist and so you can assign specific mountpoint(s) to your device(s) and thus ensure e.g. scripts will work as expected.

## What to learn
1. Overview of system mount units
2. Structure and Content of a mount unit
3. Mount at boot (immediate mount)
4. Mount at demand (mount on first access)
5. Conclusion

## 1. What is systemd mount units
---

The systemd manual has the following somewhat contradicting statement

> Mount units may either be configured via unit files, or via /etc/fstab (see [fstab(5)](http://man7.org/linux/man-pages/man5/fstab.5.html) for details). ***Mounts listed in /etc/fstab will be converted into native units dynamically at boot*** and when the configuration of the system manager is reloaded. In general, ***configuring mount points through /etc/fstab is the preferred approach***. See [systemd-fstab-generator(8)](http://man7.org/linux/man-pages/man8/systemd-fstab-generator.8.html) for details about the conversion.- [system.d.mount.5.html#FSTAB][3] (Emphasized by me)

### Logic
As mounts listed in fstab ***will be converted into native units dynamically at boot*** - logically we create the units ahead so the system don't have to bother with converting fstab - the contradiction in above statement is ***fstab is the preferred approach***. 

**What?**  Why would would it be preferable to use a configuration which must be parsed and converted when you can create native units?

**I use mount units because**
* They are easy to create
* They provide clear logic of
   - What
   - Where
   - When
   - Why
   - How
* Simple auto mount defines

### 
Using automount units to mount a device using systemd is a simple secure approach. 

The top arguments for using mount units

* you don't have to edit your fstab
* you don't have to create the mountpoint(s) beforehand
* you have a clear overview - which you will se later

This article will use an USB device which may or may not be available at boot time and when you plug it you want it to be mounted on a special path - it could be dedicated backup disk.

The basics of this guide can also be used to mount any extra devices available in your system, or remote shares e.g. WEBDAV, SMB, FTP and NFS. You can find link to a topic containing sample mount units at the end of this article.

## 2. Structure and Content of a mount unit
---
There is no rules as to where you can mount the device. Just remember a few things

* Structure
* Recognizable
* Don't use **/run** is a volatile structure recreated at boot
* Don't use **/mnt** it's primary use is temporary mount

By structure I mean organizing a directory tree - I usually recommend create a data folder on root and populate the tree with your devices and locations like below.

By recognizable I mean naming your mount point with a logical description of the content - but **don't use dashes anywhere in the mount point** - see explanation below. If you really need a double word in your data tree - use a low dash  ( **_** ). You should also avoid local accented characters (e.g. danish æ ø å Æ Ø Å)

Immediate branches are your devices - and network share mounts in a branch named for the type of fileservice e.g. NFS and SMB.

```
$ tree /data -L 2
/data
├── backup
├── build
├── nfs
│   ├── data
│   ├── music
│   ├── photo
│   ├── video
│   └── web
├── private
├── projects
├── smb
│   └── web
└── virtualbox
```

Naming the mount units uses a mandatory convention.

Lets say you have a disk for backup and the disk mounts to **/data/backup** then the filename becomes.

* **data-backup.mount**

**NOTE :** Dashes in the file name represents the directory separator - so **don't use dashes anywhere in the mount point**. 

Create a file as root
```bash
$ sudo touch /etc/systemd/system/data-backup.mount
```

Open the file with your favorite CLI editor or a gvfs compliant editor (xed or gedit)
```bash
$ xed admin:/etc/systemd/system/data-backup.mount
```

### Template
Insert the following template into the file - and modify the description - continuing our backup disk example

```text
[Unit]
Description=Mount Backup disk (/data/backup)

[Mount]
What=
Where=
#Type=
#Options=
#TimeoutSec=seconds

[Install]
WantedBy=multi-user.target
```

### Mount
The section **[Mount]** is the same data as you see in your fstab - except the **What** which must the full path to the device. The nature of systemd makes it impossible to rely on the traditional path **/dev/sda1** - so we will use **/dev/disk/by-uuid/**.

### What
To get the device uuid you can use **lsblk** command and the **-no UUID** to retrieve the uuid you need for the mount. (substitute sdy1 with your actual device and partition)

```bash
$ lsblk -no UUID /dev/sdy1
67f922cd-a61f-4d5e-84c0-ac8335a7ce67
```

Then insert the UUID into the file
```text
What=/dev/disk/by-uuid/67f922cd-a61f-4d5e-84c0-ac8335a7ce67
```

### Where
Next type in the path where you want the device mounted. Normally we would create the path beforehand as this is required for fstab - but not with systemd. If the path do not exist it will be created.

:information_source: default permissions is 755 owned by root

If your actual device is an existing device **and** the owner:group UID:GID do not match your device will be mounted read-only.

```text
Where=/data/backup
```

**Caveat using dashes in a folder-name**
Don't use a dash **-** in your path because a dash refer to a new branch on the folder tree and this will break the naming of the mount/automount file

* Invalid **/data/home-backup**
* Valid **/data/home_backup**
* Valid **/data/home\x2dbackup**

To get the correct name for your mount/automount filename when you use dashes in your folder-naming

```bash
$ systemd-escape -p --suffix=mount "/data/home-backup"
data/home\x2dbackup.mount
```

### Type
The **Type** is optional for disk devices - but from the viewpoint - the more information we provide - less work for the system. 

You can retrieve the file system the same way as the UUID.

```
$ lsblk -no FSTYPE /dev/sdy1
ext4
```
Insert the filesystem type
```text
Type=ext4
```
Other types like NFS, SMB and FTP requires **Type** to be set.

### Options
The **Options** is optional for disk devices. 

As with fstab it takes a comma separated list of options for the filesystem mounted. 

Other types like NFS, SMB and FTP requires **Options** to be set.
```text
Options=defaults,rw,noatime
```
Save the file

## 3. Mount on boot
---
This works just like your fstab - if the device is not available - the system will use the default 90 seconds for the device to be available. The behavior can be changed by setting **TimeoutSec=** to a value matching the device actual mount time or use an additional automount unit.

First reload the systemd daemon
```bash
$ sudo systemctl daemon-reload
```

Note the state of the mount
```bash
$ systemctl show -p ActiveState -p SubState --value data-backup.mount
```

Mount the device
```bash
$ sudo systemctl start data-backup.mount
```

Now you can verify the service
```bash
$ systemctl show -p ActiveState -p SubState --value data-backup.mount
```

Note that the folder was created by systemd - list the content of the mount
**NOTE:** note permissions changed to partition content
```bash
$ ls -la /data/backup
```
If the device is an internal device always available you can enable the mount and your device will be mounted during boot.

    sudo systemctl enable data-backup.mount

If - on the other hand - this is an USB device - which is not available at boot time - do not enable the the mount unit but proceed to create an `.automount` unit for your USB device.

### Check permissions
If this is a newly formatted device or a reused device - it is wise to check the permissions. If you get issues with the device being mounted read-only - depending on filesystem - it may be due to the permissions stored on the device.

If the device is using a Linux file system you can change permissions when it is mounted. To change the permissions on your games partition mounted at **`/data/games`**

    sudo chmod ugo+rw /data/games -R

If your device is a shared device dual booting Windows and Linux and formatted using NTFS and it mounts read-only - reboot your system into Windows and disable features like Fastboot, hibernation, sleep and hybrid sleep as these will taint the filesystem and cause Linux to mount it read-only.

## 4. Mount on demand
---
An automount unit is a special unit which takes care of activating the corresponding mount unit when the mount path is accessed. The difference for mount and automount is **when** it is mounted.

To demonstrate the automount - first unmount the data-mount
```bash
$ sudo systemctl stop data-backup.mount
```

Verify  the content of the mount point - it should now be empty.
```bash
$ ls -la /data/backup
```

Now we proceed to create an automount for the mount. Create a file - it must be named using the same rule as the mount but with the extension of **.automount**
```bash
$ sudo touch /etc/systemd/system/data-backup.automount
```
**NOTE:** the automount service will fail if the device path is mounted

### Automount define
Edit the file and insert the following content and save it.
```text
[Unit]
Description=Automount backup partition

[Automount]
Where=/data/backup
TimeoutIdleSec=10

[Install]
WantedBy=multi-user.target
```

### Test automount
---
Now enable and start the automount for data-backup
```bash
$ sudo systemctl enable --now data-backup.automount
```

You know you unmounted the device - don't mount it - just access the **/data/backup** folder from terminal or a file manager - you will find systemd mounts it behind the scenes.

## 5. Conclusion
---
This approach is a safer way than editing your fstab. Worst case the device is not mounted - if you make a mistake in fstab - the system breaks.

You can mount all kind of devices - NFS, SMB, FTP, WEBDAV, SFTP, SSH - using this recipe.

[1]: https://wiki.archlinux.org/index.php/Udev
[2]: https://wiki.archlinux.org/index.php/Udev#Setting_static_device_names
[3]: http://man7.org/linux/man-pages/man5/systemd.mount.5.html#FSTAB
