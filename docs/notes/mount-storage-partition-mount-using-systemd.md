---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Store personal files using separate partition (systemd)'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

**Difficulty: ★★☆☆☆**

## Storing personal files on different partition

Some Linux users like to experiment with their system and more often than not it leads to the decision of reinstalling the system. That repeatedly creates the issue on how to handle the home folder with configurations and personal files.

The purpose of this document is to create an easy and convenient way of safeguarding personal data files while retaining the home folder.

## Preparation

http://www.pathname.com/fhs/pub/fhs-2.3.html#INTRODUCTION states

! The FHS document has a limited scope:
! * Local placement of local files is a local issue, so FHS does not attempt to usurp system administrators.
! * FHS addresses issues where file placements need to be coordinated between multiple parties such as local sites, distributions, applications, documentation, etc.

So for the sake of this HowTo there is no standard as we are talking about 

***Local placement of local files is a local issue***.

Your system would ideally have two disks but if that is not possible - laptops come to mind - then a system with at least three partitions or more will suffice.

1. EFI System Partition (ESP)
2. Root with home
3. User data

## Preparations
Create a mount unit for the partition. For the sake of the guide we will use a non-existing device name - which you must substitute with your actual system. First list your devices - this may yield an output similar to this

```bash
➜  ~ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdy           8:0    0 465,8G  0 disk 
├─sdy1        8:1    0 419,2G  0 part
└─sdy2        8:2    0  46,6G  0 part 
sr0          11:0    1  1024M  0 rom  
nvme0n9     259:0    0 476,9G  0 disk 
├─nvme0n9p1 259:1    0  1000M  0 part 
├─nvme0n9p2 259:2    0   459G  0 part /
└─nvme0n9p3 259:3    0    17G  0 part [SWAP]
```
Using the sample output we will use sdy1 as the target.

Because `udev` lists devices completely arbitrarily you cannot rely on the *sdy* namespace - we need the UUID for our mount - and the easiest way to obtain it in a usable manner is to pipe it into a file - then edit the file.

Use this command to create the mount unit in your home then edit the file with your favorite editor.

**IMPORTANT**: For systemd the mount point must match the filename
> Examples
> * /data/private = data-private.mount
>* /data/private/music = data-private-music.mount
> * /data/development/python = data-development-python.mount

```bash
lsblk -no UUID /dev/sdy1 > ~/data-private.mount
```
Modify the file to read like this - `$UUID` - is only a placeholder for the result on your system - modify accordingly
```bash
➜  ~  cat ~/data-private.mount 
[Unit]
Description=Mount private partition

[Mount]
What=/dev/disk/by-uuid/$UUID
Where=/data/private
Type=ext4
Options=defaults,rw,noatime

[Install]
WantedBy=multi-user.target
```
Now move the file to */etc/systemd/system* and change owner to root.
```bash
$ sudo mv ~/data-private.mount /etc/systemd/system
$ sudo chown root:root /etc/systemd/system/data-private.mount
```
Start and enable the mount unit
```bash
$ sudo systemctl enable --now data-private.mount
```
If the partition is newly created then you need to set permissions on the mount to allow for your username to be able to write to the partition.
```bash
$ sudo chmod ugo+rwx /data/private
```
Or you could change the ownership of the partition to your username
```bash
sudo chown $USER:$USER /data/private
```

## Link your data
Move the data to the new location - an example could be your music collection in the ~/Music folder.
```bash
$ mv ~/Music /data/private
```
Now create a symlink in your home the new location
```bash
$ ln -s /data/private/Music ~/Music
```
The process is a logic target for a script to handle. Create a new folder to hold some scripts
```bash
$ mkdir /data/private/scripts
```
Copy the mount unit to the script folder
```bash
$ cp /etc/systemd/system/data-private.mount /data/private/scripts
```
Create a new file **/data/private/scripts/20-link-data.sh** and make it executable.
```bash
$ touch /data/private/scripts/20-link-data.sh
$ chmod +x /data/private/scripts/20-link-data.sh
```
Edit the file and and paste the following content into it. Modify the script and extend it to match your system and the folders you want to control and save the file. 

```bash
#!/bin/bash
echo "Moving and linking folders"
cd $HOME

# moving and linking documents
mv -i Music /data/private
ln -s /data/private/Music Music
```

The script is designed to be run on your home folder or a newly created home where no files has yet been saved. 

A simple safeguard exist so you must verify the moving of the folders - if they exist.

## Data structure
Mount units creates the mount point automagically when started and the mountpoint is not yet existing.

Create a new file in your `/data/private/scripts` folder and make it executable

```bash
$ touch /data/private/scripts/10-mount-units
$ chmod +x /data/private/scripts/10-mount-units
```

Edit the file and paste the following content into it. Modify the script to match your system and save the file.

```bash
#!/bin/bash
sudo cp ./data-private.mount /etc/systemd/system
sudo chown root:root /etc/systemd/system/data-private.mount
sudo systemctl enable --now data-private.mount
```

## Conclusion
With these two scripts - you can have your data mounted and available with the few seconds these will take to run.

The scripts folder can be copied to a USB stick and run from it or be kept in the data partition and run from there.