---
title: 'Samba Quick Setup public read-write shares'
date: '11:12 05-02-2023'
taxonomy:
    category:
        - docs
---

# Linux, Samba and NTFS

!! The following is highly insecure as it allows any and all devices on the network - **including ransomware** - to access and write/rewrite your data - so be warned.

Documenting how to handle NTFS formatted devices shared using Samba

## Mounting NTFS formatted devices

On a designated server - open a terminal and switch to root context
```
su -l root
```
Create the following folders
```
mkdir -p /srv/samba/ntfs /srv/samba/public
```
Using the long listing format - list the content of /srv/samba - pay attention to the folder permissions
```
# ls -l /srv/samba
drwxr-xr-x 2 root root 4096 Feb  4 07:14 ntfs
drwxr-xr-x 2 root root 4096 Feb  5 07:55 public
```

Create a mount unit - mounting your ntfs formatted device in /srv/samba/ntfs (this is a documenthere so simply copy and paste into the terminal and press enter - use the nice copy button to right of the textbox)

You must modify the first line to use the correct path for your device
```
MYDEVICE_UUID=$(lsblk -no uuid /dev/sda1)
cat << EOF > /etc/systemd/system/srv-samba-ntfs.mount 
[Unit]
Description=Old Win Device

[Mount]
What=/dev/disk/by-uuid/$MYDEVICE_UUID
Where=/srv/samba/ntfs
Type=ntfs
Options=defaults,rw,noatime

[Install]
WantedBy=multi-user.target
EOF
```
Start and enable the unit
```
systemctl enable --now srv-samba-ntfs.mount
```
Once again - using the long listing format - list the content of /srv/samba - pay attention to the ntfs folder permissions
```
# ls -l /srv/samba
total 8
drwxrwxrwx 1 root root 4096 Feb  4 08:18 ntfs
drwxr-xr-x 2 root root 4096 Feb  5 07:55 public
```
What we learned from this little exercise it that ntfs - just like vfat, exfat, fat32 et al. is a permissionless filesystem and when mounted any and all users on the system may write to the device with no regard whatsoever to the ownership and permissison on the mountpoint.

## Samba and NTFS
On the designated server - install **samba** and **avahi** packages 
```
pacman -Syu samba avahi --needed
```
Create the samba configuration file as follows (this is a documenthere so simply copy and paste into the terminal and press enter  - use the nice copy button to right of the textbox).
```
cat << EOF > /etc/samba/smb.conf
[global]
   workgroup = MANJARO
   server string = Manjaro Samba Service
   server role = standalone server
   log file = /var/log/samba/log.%m
   max log size = 50
   guest account = nobody
   map to guest = Bad Password

   min protocol = SMB2_02

   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *New*UNIX*password* %n\n *ReType*new*UNIX*password* %n\n *passwd:*all*authentication*tokens*updated*successfully*
   pam password change = yes

[public]
   path = /srv/samba/public
   public = yes
   writable = yes
   printable = no

[ntfs]
   path = /srv/samba/ntfs
   public = yes
   writable = yes
   printable = no

EOF
```

Then enable and start the services
```
systemctl enable --now smb avahi-daemon
```

## Client access to Samba server
On your client install the packages **smbclient** and **avahi**
```
sudo pacman -Syu smbclient avahi --needed
```
Start the avahi service
```
sudo systemctl enable --now avahi-daemon
```
Open your file manager -> Open the path navigator using <kbd>Ctrl</kbd>+<kbd>L</kbd> and input the address of your samba service.
```
smb://ip.x.y.z
```
Open the ntfs folder and create a new document

## Accessing Samba share from Windows



## That's all folks
You didn't think it could be that simple did you?
