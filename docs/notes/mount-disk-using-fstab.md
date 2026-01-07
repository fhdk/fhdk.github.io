---
title: 'Permanent mounts using fstab'
taxonomy:
    category:
        - docs
---

## Make a permanent mountpoint.

http://www.pathname.com/fhs/pub/fhs-2.3.html#INTRODUCTION states

> The FHS document has a limited scope:
>     Local placement of local files is a local issue, so FHS does not attempt to usurp system administrators.
>     FHS addresses issues where file placements need to be coordinated between multiple parties such as local sites, distributions, applications, documentation, etc.

So for the sake of this HowTo there is no standard as we are talking about ***Local placement of local files is a local issue*** and the following is therefore to be seen as a suggestion not a requirement. You are the user/owner of the system and you should choose whatever structure is logical to you.

### Location considerations
If you are the one and only user  of the system - i.e. it is _not_ a shared system - you can create your mount points in your home folder - nothing wrong. But if it is a shared system you should create the mount structure beginning from the root e.g. `/data/name-of-mount` or for removable devices `/media/media-name`.

The `/mnt` is a predefined location for temporary mounts - so avoid using it for permanent mounts - it can go oh-so-wrong.
Also avoid the `/run` location as `/run` is a volatile structure created as a tmpfs and will be removed at reboot or shutdown.

The following will use a substitute name for the root `$mount`  mount e.g. `/home/username` or `/data` so replace with your `$mount` with whatever you choose.

### Locate the device and partition(s).

    $ lsblk
    sda      8:0    0 223,6G  0 disk 
    ├─sda1   8:1    0   512M  0 part 
    ├─sda2   8:2    0    60G  0 part 
    ├─sda3   8:3    0   130G  0 part 
    └─sda4   8:4    0  33,1G  0 part [SWAP]
    sdb      8:16   0   477G  0 disk 
    └─sdb1   8:17   0 419,1G

Depending on your computers hardware you might get other device names and partition numbers. Modify the following according to your output.

### The `/data` folder
To help yourself locating your folders easily you should use a consistent approach across your systems. When you share partitions across users it is preferable to use a location outside the /home structure and thus avoiding permissions issues across users.

### The `/media` folder
Systemd uses a **media** folder located under `/run` to mount removable medias  on the fly. The root folder `/media` is a folder commonly used for mounting removable media on a defined structure. Ubuntu and other distros used it and certain automount utilities use the folder.

#### Permissions
If you define the mount outside your home e.g. `/data` you must also define the access control for the folder. ACL is a topic on its own so to keep it simple just set the permissions so everyone can use it. This is _not_ necessary if you create the mount point inside your home

    sudo chmod ugo+rwx $mount

### Make a permanent mountpoint.
Create a folder for the purpose (change all references to **mydisk** as you would want a more descriptive name).
If you have several partitions you should name the folders with decriptive names - what ever you feel appropriate :)

    $ mkdir $mount/mydisk

### Mount the partition.
Replace the ***Xy*** with the device and the partition number you found.

    $ sudo mount /dev/sdXy $mount/mydisk

### Mount partition on boot (not removable media)
To mount it permanently on boot modify your `/etc/fstab` to include the partition on boot.

    $ su -
    # echo "UUID=$(lsblk -no UUID /dev/sdXy) $mount/mydisk $(lsblk -no FSTYPE /dev/sdXy) defaults,noatime 0 2" >> /etc/fstab
    # mount -a

If your drive has more than one partition repeat above steps as needed adjusting for each partition.

### The echo command looks cryptic right? 
It is not - try this

    $ lsblk -no UUID /dev/sdb1                                                 
    bogus-id-4e9e-403d-bbd0-72e62301e07c

This outputs the unique identifier of your partition. This is used in fstab to ensure the same partition is always mounted the right places. Now try this

    $ echo $(lsblk -no UUID /dev/sdb1)                                         
    bogus-id-4e9e-403d-bbd0-72e62301e07c

You see? We got the exact same result but with this approach we include it in a command which writes a lot more info. Note that we can use a similar construction to get the filesystem type for the partition.

    $ echo $(lsblk -no FSTYPE /dev/sdb1)
    ext4

To write this to a file we use a pipe - an arrow pointing to a file name.

    $ echo "UUID=$(lsblk -no UUID /dev/sdb1) $mountmydisk $(lsblk -no FSTYPE /dev/sdXy) defaults,noatime 0 2" > test.txt

To avoid deleting the contents of a file we **append** to it with double arrow

    $ echo "UUID=$(lsblk -no UUID /dev/sda2) $mount/myhome $(lsblk -no FSTYPE /dev/sdXy) defaults,noatime 0 2" >> test.txt

Have a look at the file

    $ cat test.txt
    UUID=bogusid1-4e9e-403d-bbd0-72e62301e07c $mount/mydisk ext4 defaults,noatime 0 2
    UUID=bogusid2-314f-40a9-b82f-a8fcfabbccb4 $mount/myhome ext4 defaults,noatime 0 2

## Network locations and removable devices
For network locations and removable devices which needs a specific location e.g. a backup device targeted in scripting read the article on using systemd mount units.

* https://forum.manjaro.org/t/root-tip-use-systemd-to-mount-any-device/1185
* https://forum.manjaro.org/t/howto-systemd-mount-unit-samples/1191

## More reading
* https://forum.manjaro.org/t/tutorial-understanding-and-working-with-unix-filesystems-and-permissions/65793
