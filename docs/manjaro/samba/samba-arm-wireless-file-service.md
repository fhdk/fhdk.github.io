---
title: 'ARM wireless Samba file service based on Arch'
date: '07:32 14-10-2022'
published: true
taxonomy:
    category:
        - docs
publish_date: '16-10-2022 12:11'
---

Difficulty: ★★★★☆

## Wireless Raspberry Pi file server

### Goal

The exercise is to use use Samba to share different content for different audiences using an Arch Linux based OS.

Which one you choose is up to you - for this writeup I am using Manjaro ARM minimal.

The bar is set high as the installation and configuration is done from scratch using a SSH terminal session to the target system.

It will also require understanding of the Access Control List in a GNU/Linux system as well as networking concepts.

The final setup will provide the following features

Users and groups
* user:admin group:admin home:admin
* user:hans group:users,office (no-shell, no-home)
* user:grete group:users,admin (no-shell, no-home)

Shares
* public share
   * authenticated users can edit files
   * guests can read files
* document share
   * stored on local root partition accessible to authenticated users
   * files may be edited by users belonging to the *admin* group
   * *office* users can only read the documents
* music library (administrated by the dedicated admin)
   * stored on external partition - in this case an USB stick

### Prerequisite

* Reference [Arch Wiki][2] on Samba
* Arch (based) Workstation
* The following packages **micro**, **smb-client**, **gvfs** and **gvfs-smb**
* The package **rpi-imager** which you either install from the Manjaro repo or build from AUR
* A cardreader to be able to write the image to a micro sd-card
* Raspberry Pi 4
* Micro sd-card

Do a full system sync adding the packages  to your workstation (remember to reboot if your kernel is updated)
```
sudo pacman -Syu micro smb-client gvfs gvfs-smb base-devel git --needed
```
If you are using Manjaro - install rpi-imager from the repo
```
sudo pacman -S rpi-imager
```

If using EndeavourOS or Arch you must build the rpi-imager from AUR
```
git clone https://aur.archlinux.org/rpi-imager
cd rpi-imager
makepkg -is
```

Plug your sd-card in the card reader attached to your workstation

The RPI imager is simple to use. Upon selection of **source** and **storage** the app downloads the image, writes to sd-card and verifies the image - ready to use.

Just launch the app - navigate to locate the Manjaro ARM minimal and write it to your sd-card.

Create the [wifi-hack.sh script][1] and use it to configure your PI's WiFi.

The script takes 6 arguments `<blockdev> <ssid> <passphrase> <cidr> <gateway> <dns>`.

The following is an example - amend the arguments for your use case
```
sudo hack-wifi.sh /dev/mmcblk0 my-ssid 192.168.1.90/24 192.168.1.1 192.168.1.1
```

Move the sdcard to your pi and power up. Wait for the device to settle - the green activity led becomes quiet - more important - allow the device to connect to your wifi.

We will switch back and forth and forth between ssh and workstation - and we will be referencing the pi's smb service quite a lot.

So to avoid having to type the IP address repeatedly we will create an alias for the configured IP and append to your workstation's `hosts` file.

For this writeup I have chosen **rpi**
```
echo "192.168.1.90 rpi" | sudo tee -a /etc/hosts
```
Connect to the pi using the either the address or the alias

```
ssh root@rpi
```
The oem installer script will prompt you for a hostname - be smart and use the alias you just created.

After completion of the script the device reboot.

Reconnect to the device using the newly configured user.

```
ssh username@pi
```
Configure the mirrorlist and do a full sync while adding **samba**, **firewalld**, **micro**.
```
sudo pacman-mirrors --continent && sudo pacman -Syyu samba micro firewalld
```
Then enable firewalld and configure use firewall-cmd to configure the service for the home zone
```
sudo systemctl enable --now firewalld
sudo firewall-cmd --set-default-zone=home
sudo firewall-cmd --permanent --add-service={samba,ssh} --zone=home
```
After sync has completed reboot your rpi and reopen the ssh connection
```
sudo reboot
```

## data tree
On the pi we will store data in a designated tree starting from our system root (**/**)
```
mkdir -p /data/office
mkdir -p /data/media/music
mkdir -p /data/public 
```
Visualised using `tree`
```
# tree /data
/data
├── media
│   └── music
├── office
└── public
```
### Permissions
#### office
The office folder's permission should be editable by group with no access for other - including guests.
```
chmod go+w,o-rx /data/office
```
#### media
The content of **/data/media/music** is stored on an external USB device, so initially there is no data. The device may or may not be attached, we will handle that later as this pose a challenge.

The media folder has suitable permissions - no need to change.
#### public
The public folder is writable by group and readable for other
```
chmod g+w /data/public
```
The **office** group doesn't exist - so let's create it and assign the group to the documents folder.
```
groupadd office
chgrp office /data/office
```
For the **public** folder we decided that authenticated users are allowed to write - guests are not. The default **users** group is perfect for this.
```
chgrp users /data/public
```
If you list the content of the data folder vs. the bind mounted folders you will note the permissions reflect the permissions set on the data folder.
```
# ls -l /data
drwxr-xr-x 3 admin admin  4096 Aug 10 01:29 media
drwxr-xr-x 2 admin office 4096 Oct 16 10:40 office
drwxrwxr-x 2 admin users  4096 Oct 16 09:48 public
````
Listing `/srv/samba` and `/srv/samba/data` folder
```
# ls -l /srv/samba
drwxr-xr-x 5 root  admin 4096 Oct 16 10:40 data
drwxrwxr-x 2 admin users 4096 Oct 16 09:48 public
# ls -l /srv/samba/data
drwxr-xr-x 3 admin admin  4096 Aug 10 01:29 media
drwxr-xr-x 2 admin office 4096 Oct 16 10:40 office
drwxrwxr-x 2 admin users  4096 Oct 16 09:48 public

```

### data protection
To ensure we do not accidental enable folder traversel outside the shares we will use **bind** mounts.

Mounting using **bind** is a simple and efficient method of making data from one folder available from another folder on the same system. The function is similar to symlink but you cannot create folder traversal using dots.

We will create some corresponding folders in **/srv** and bind our data there.
```
mkdir -p /srv/samba/data
mkdir -p /srv/samba/public
```
Open the file /etc/fstab with micro
```
micro /etc/fstab
```
Append the following lines
```
/data             /srv/samba/data none bind 0 0
/data/public      /srv/samba/public none bind 0 0
```
Now execute a reload and mount everything
```
systemctl daemon-reload
mount -a
```
When we list the **/srv/samba** we see how our changes to the data folders is reflected in share tree
```
# ls -l /srv/samba
drwxr-xr-x 5 root  admin 4096 Oct 16 10:40 data
drwxrwxr-x 2 admin users 4096 Oct 16 09:48 public
# ls -l /srv/samba/data
drwxr-xr-x 3 admin admin  4096 Aug 10 01:29 media
drwxr-xr-x 2 admin office 4096 Oct 16 10:40 office
drwxrwxr-x 2 admin users  4096 Oct 16 09:48 public
```

And the tree
```
/srv/samba
├── data
│   ├── media
│   │   └── music
│   ├── office
│   └── public
└── public
```

## Basic samba configuration
In your shell session switch to root session - so we don't have to prepend everything with sudo.
```
su -l root
```
Locate the samba configuration folder on your pi
```
cd /etc/samba
```
Samba is always configured from scratch - there is no default config, it must be created before you can start the service.

The micro editor is terminal editor with support for copy/paste and other well know editor shortcuts.

In the samba configuration folder start micro and create the required configuration **smb.conf**

```
micro smb.conf
```

We start by defining two sections **global** configuration and **public** share configuration

```
[global]
   workgroup = WORKGROUP
   server string = Samba File Server
   server role = standalone server
   log file = /var/log/samba/log.%m
   max log size = 50
   guest account = nobody
   map to guest = Bad Password
   
   min protocol = SMB2   # Think ransomware - you don't wannacry

[public]
   path = /srv/samba/public
   public = yes
   writable = yes
   printable = no

[data]
  path = /srv/samba/data
  public = no
  writable = yes
  printable = no
```
Run the command `testparm` to ensure your smb.conf is free from spelling errors then start the service.
```
testparm -s
```
If you are curios about smb.conf - try this

```
testparm -s -v > ~/smb-defaults.txt | micro 
```

### public share

Verify the public share by opening the file manager on your workstation.

Use **Ctrl+L** and input `smb://ip.x.y.z/public` then press **Enter**

Login using the anonymous method. Try creating new file - you should expect access denied.

Then go back to your shell and create an empty text file in public folder. 
```
touch /data/public/test.txt
```
Go back to your file manager press **F5** to reveal the new file.

All good - now unmount the public share from the file manager and reopen the share - this time login using the username and credentials from your ssh session - this time edit the file and save it.

Did you get access denied once more? Good - this is expected behavior as there is no users in the samba trust database. Unmount the public share using the eject button in the file manager.

## user - samba vs. system
**It is very important to remember**

- a system user is not a samba user
- a samba user requires a system user

If you change your user password you either remember two passwords or change using `smbpasswd` for your username.

To enable passwd sync maintenance from samba to system - add this to the global section of `smb.conf`
```
[global]
  ....
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *New*UNIX*password* %n\n *ReType*new*UNIX*password* %n\n *passwd:*all*authentication*tokens*updated*successfully*
   pam password change = yes
  ....
```
Any changes to smb.conf requires the service to be restarted and as a precaution always test the config before restarting.

Using this command will prevent samba from restarting if the testparm command fails
```
testparm && systemctl restart smb
```

### anonymous user
**Anonymous users is a security risk.** 
You have no control over the contenton your share. This is bad - very, very bad - think ransomware - you don't *wannacry* :sob:  If you - despite all warnings - must create a public share with no restrictions refer to [Samba Usage][3] on Arch Wiki.

### authenticated user
Remember the leading sentence for this section - **a system user is not a samba user**. Since our service should allow authenticated users to create content in the public folder - let us start easy with your ssh username.
```
smbpasswd -a your-user
```
In your file manager mount the share again - this time using your ssh user and the samba password. Open the empty file - enter some text and save it. Try creating some new content on the public share.

This time you are authenticated and thus allowed to make changes.

Remove your test content from the share and unmount.


### creating users - add to samba
Our server is a file server only and as such - besides the admin - users has no use for home folder and they don't need shell access.

Our challenge includes and administrator and two distinct authenticated users with access to the documents folder.

* *users* group can read/write public content
* *office* group can read documents
* *admin* group can read/write all content

As we configured samba using **unix password sync = yes** samba will sync the password back to the system.

#### creating users
A designated administrator of shared content. The user has system login permissions and a home folder
```
useradd --create-home --user-group --groups office,users --shell /bin/bash admin
smbpasswd -a admin
```
A user named *hans* in group *office* with no shell, no home and no personal group 
```
useradd --no-create-home --no-user-group --groups office  -s /bin/nologin hans
smbpasswd -a hans
```
Let's create the office user - name *grete* with no shell, no home, no personal group but a supplementary group *office*
```
useradd --no-create-home --no-user-group --groups admin --shell /bin/nologin grete
smbpasswd -a grete
```
### admin user permissions
When you list the content of the `/data` folder, `root` is listed as owner - but we have our dedicated admin.

Let's change the group for */data* root  to admin - allowing admin to create new folders
```
chgrp admin /data
```

Next we change the owner of content inside */data/* - you did remove the test files - right? Otherwise the owner will change for those as well.
```
chown admin /data/* -R
```

## the data share
Edit your samba configuration and add the following section

```
[data]
  path = /srv/samba/data
  public = no
  writable = yes
  printable = no
  guest ok = no
```
Test the config

```
testparm -s
```
restart the service

```
systemctl restart smb
```

On your workstation open a terminal and run

```
smbclient -L rpi -U admin
Password for [WORKGROUP\admin]:

        Sharename       Type      Comment
        ---------       ----      -------
        public          Disk      
        data            Disk      
        IPC$            IPC       IPC Service (Manjaro Samba Server)
SMB1 disabled -- no workgroup available

```

## the media share
The media share resides on a removable device and we want the device to be mounted all times. The problem with removable devices is - they are removable - which is a challenge to be solved.

We will face this challenge using systemd units.

We need the UUID of the partition we want to mount - so attach the device to the Raspberry Pi and list the available block devices

```
# lsblk -A
lsblk -A
NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda            8:0    1  28,8G  0 disk 
└─sda1         8:1    1  28,8G  0 part 
mmcblk0      179:0    0  14,6G  0 disk 
├─mmcblk0p1  179:1    0 457,8M  0 part /boot
└─mmcblk0p2  179:2    0  14,1G  0 part /srv/samba/public
                                       /srv/samba/data
                                       /
```
The partition is /dev/sda1 - let's get the UUID and pipe it to the unit file.
```
lsblk -no UUID  /dev/sda1 > /etc/systemd/system/data-media-music.mount
```
systemd mount unit naming convention is to use the mountpoint as filename replacing **/** with **-**.

In our case we have placed the music files in the folder **/data/media/music** so this is where we mount the partition.

Open the unit file using micro
```
micro /etc/systemd/system/data-media-music.mount
```
Alter the content to match below - arrange for you UUID which is unique for my system to end the line *What=/dev/disk/by-uuid/*. (do not add spaces before or after the equal sign **=** and observe the casing of properties)
```
[Unit]
Description=USB stick with music files

[Mount]
What=/dev/disk/by-uuid/some-uuid-from-yoursystem
Where=/data/media/music
#Type=
Options=defaults.rw.noatime

[Install]
WantedBy=multi-user.target
```
Now we create unit which takes care of mount the partition when it is needed. The same rule on filename applies - but the extension is *.automount*
```
micro /etc/systemd/system/data-media-music.automount
```
The content of the automount is as follow
```
[Unit]
Description=USB stick with music files
ConditionPathExists=/data/media/music

[Automount]
Where=/data/media/music
TimeoutIdleSec=300

[Install]
WantedBy=multi-user.target
```
All that is left now is to enable the automount unit

```
systemctl enable --now data-media-music.automount
```
To share the music folder on a common sharepoint - edit your smb.conf and add the following section 
```
[music]
  path = /srv/samba/data/media/music
  public = yes
  printable = no
  guest ok = yes
```
Test your configuration  
```
testparm -s
```
Restart the service
```
systemctl restart smb
```


## Done

You have setup your raspberry pi to run a samba service and succesfully shared a public and a private folder.

Also posted at [Manjaro Forum][4]


[1]: https://root.nix.dk/en/utility-scripts/manjaro-arm-setup-wifi-script
[2]: https://wiki.archlinux.org/title/Samba
[3]: https://wiki.archlinux.org/title/Samba#Usage
[4]: https://forum.manjaro.org/t/root-tip-how-to-samba-fileserver-over-wifi-using-raspberry-pi/124428