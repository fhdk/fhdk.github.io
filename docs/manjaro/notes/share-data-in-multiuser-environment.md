---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Sharing data in multiuser environment'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

# Data sharing

Some Linux users like to experiment with their system and more often than not it leads to the decision of reinstalling the system. 

For the purpose of increased usability, uptime or other reasons, some like to have several systems installed.

That repeatedly creates the issue on how to handle not only the home folder but also data you would like to share between distributions or multiple users of the same system - that can be configurations and/or personal files.

The purpose of this document is to create an easy and convenient way of safeguarding or sharing personal data files while retaining the home folder.

Everything in this document is meant as a starting point for creating your own adapted version. It is not one-size-fits-all but it should give you - the reader - an idea on how to achieve the goal of protecting user data during re-installation and restore these without extensive copy operations.

Besides the above it is intended to provide a reasonable simple way of sharing data between users or systems. This guide is a modified version of the guide detailing how to [store personal data on a different partition](https://archived.forum.manjaro.org/t/howto-move-your-personal-data-to-different-partition/47790?u=linux-aarhus).

## Preparation
One comment I always hear is *you can't make folders outside the folder standards* so here is the [link to the FHS definition](http://www.pathname.com/fhs/pub/fhs-2.3.html#INTRODUCTION) and I quote

> The FHS document has a limited scope:
>     Local placement of local files is a local issue, so FHS does not attempt to usurp system administrators.

So to emphasize **Local placement of local files is a local issue**.

Your system would ideally have two disks but if that is not possible - laptops come to mind - then a custom partitioned system with three or more partitions partitions will suffice.

As we will be moving folders which traditionally are those taking most space like  ***Documents*, *Downloads*, *Music*, *Pictures*, *Video*** we will be keeping the user's home folder as the standard location */home/$USER*

You can easily do this without reinstalling - suffice you have enough diskspace to resize your root partition. **Don't touch your $esp partition** - it will probably make your system unbootable.

1. Efi System Partition (ESP)
2. Root with home
3. User data

Ensure the data partition is formatted with a filesystem which can be read by the systems accessing the partition. The general recommendation in this context is ext4. If you want to share files between Windows and Linux format the partition with exFAT.

If you are not reinstalling then begin with resizing either the root partition or another suitable partition.

If you choose to resize your root partition - you need reboot your system on a live media with the suitable software as a mounted partition cannot be shrinked only extended.

## Setting up the partition

### Mount point
Create a mount point in the root of the file system and set permissions.

    sudo mkdir /data/shared
    sudo chmod ugo+rwx /data/shared
    
Next step is to mount the partition and modify fstab to mount it at boot. The simple and preferred method is to use the **Gnome Disk Utility** and modify the mount options so the partition mounts at boot time in the **/data/shared** folder.

A suitable set of mount options is

    defaults,rw,noatime

### Foldernames in this document
This document will assume xdg-user-dirs english foldernames. You can of course adapt it entirely to your preferences. So lets move to creating folders.

### Creating folders
Ensure your partition is mounted at the designated mountpoint before continuing.

Create the folders you want to share between systems or between users e.g. for Music, Pictures and Documents.

    mkdir /data/shared/Music /data/shared/Pictures  /data/shared/Documents

The */data/shared* folder is writeable by everyone so if you are maintaining a multi user system you can further adapt the folder structure by creating a $USER folder with permissions for the user only e.g.

    sudo mkdir /data/shared/$USER
    sudo chown $USER:$USER /data/shared/$USER
    sudo chmod u+rwx,go-rwx /data/shared/$USER

## Create the mount script

Assuming you have created a symlink to the shared folder named *Data* we create folder to hold our scripts

    $ mkdir ~/Data/scripts

Copy your `/etc/fstab` to the scripts folder

    $ cp /etc/fstab ~/Data/scripts/fstab.txt

Edit the file `fstab.txt` and remove everything except the line which mounts your data partition.

Proceed with creating a script in the scripts folder and make it executable

    $ touch ~/Data/scripts/20-mod-fstab
    $ chmod +x ~/Data/scripts/20-mod-fstab

Edit the script file and paste the following content into it

```
#!/bin/bash
scriptdir=${PWD}
protected=$(grep 'PROTECT' /etc/fstab)
if ! [[ -z $protected ]]; then
	echo ""
	echo "Already modified! exit ..."
	exit
fi
echo "Modifying a copy of /etc/fstab -> ${scriptdir}/fstab.org ..."
cp /etc/fstab ${scriptdir}/fstab.org
echo "# PROTECTION against double merge" >> ${scriptdir}/fstab.org
echo "Merging `fstab.txt` into `fstab.org`"
cat ${scriptdir}/fstab.txt >> ${scriptdir}/fstab.org
echo "Using sudo: Replacing /etc/fstab ..."
sudo mv -bi ${scriptdir}/fstab.org /etc/fstab
sudo chown root:root /etc/fstab
echo "Using sudo: Mounting /etc/fstab"
sudo mount -a
echo "Done!"
```

This script is a convenience script to be used when ever you have reinstalled your system and you you need to re-establish the mount point. **The script expects the mount folder to be present**.

## Link your data

Depending on the folders you want to control you must ceate a symbolic link in your home to the new location. 

Decide which folders you want to control and create a new file in your `~/Data/scripts` folder and make it executable

    $ touch ~/Data/scripts/30-link-data
    $ chmod +x ~/Data/scripts/30-link-data

Edit the file and and paste the following content into it. Modify the script to match your system and the folders you want to control and save the file. If want bulletproof folder structure consider the [information here](https://archived.forum.manjaro.org/t/wiki-move-your-personal-data-to-different-partition/47790/8?u=fhdk)

```
#!/bin/bash
echo "Moving and linking folders"
cd ~

# moving and linking documents
mv -i Documents ~/Data
ln -s ~/Data/Documents

# moving and linking music
mv -i Music ~/Data
ln -s ~/Data/Music

# moving and linking pictures
mv -i Pictures ~/Data
ln -s ~/Data/Pictures
```

The script is designed to be run on your home folder or a newly created home where no files has yet been saved. 

A simple safeguard exist so you must verify the moving of the folders - if they exist.

## Data structure

Create a new file in your `~/Data/scripts` folder and make it executable

    $ touch ~/Data/scripts/10-create-data-structure
    $ chmod +x ~/Data/scripts/10-create-data-structure

Edit the file and paste the following content into it. Modify the script to match your system and save the file.

```
#!/bin/bash
datadir="/home/$USER/Data"
echo "Creating Data symlink"
ln -s /data/shared ${datadir}
```

The script is used to create your desired data structure. Add what you deem necessary.

## Conclusion

You have now created three scripts and a text file which can be used whenever you have reinstalled your system.

Just run the scripts in sequence 10- 20- 30- and your data is mounted, linked and available - all within minutes.

The scripts folder can be copied to a USB stick and run from it or be kept in the data partition and run from there.


## Reference files

[details="mounting details"]
```
/data/private/scripts >>> cat fstab.txt                                         

UUID=456bf27c-7e14-48b4-9df8-a49254dcf79e /data/projects   ext4 defaults,noatime 0 2
UUID=899b1bc4-219c-4b92-8539-0aa118d9780a /data/build      ext4 defaults,noatime 0 2
UUID=1f3d1a6e-b4b5-46da-909a-8a87a45dd18b /data/private    ext4 defaults,noatime 0 2
#UUID=67f922cd-a61f-4d5e-84c0-ac8335a7ce67 /data/virtualbox ext4 defaults,noatime 0 2
UUID=a27d7c58-0ebb-4c93-bcec-f9387f7d34f6 /data/virtualbox ext4 defaults,noatime 0 2
#UUID=b925148a-0729-447a-ad95-4d9e27dcb442 /data/usb/seagate ext4 defaults,noatime 0 2

tmpfs   /var/spool tmpfs   defaults,noatime,mode=1777   0  0
tmpfs   /var/tmp   tmpfs   defaults,noatime,mode=1777   0  0

diskstation:/volume1/data   /data/nfs/data  nfs auto,x-systemd.automount,x-systemd.device-timeout=10,timeo=14,x-systemd.idle-timeout=1min 0 0
diskstation:/volume1/music  /data/nfs/music nfs auto,x-systemd.automount,x-systemd.device-timeout=10,timeo=14,x-systemd.idle-timeout=1min 0 0
diskstation:/volume1/photo  /data/nfs/photo nfs auto,x-systemd.automount,x-systemd.device-timeout=10,timeo=14,x-systemd.idle-timeout=1min 0 0
diskstation:/volume1/video  /data/nfs/video nfs auto,x-systemd.automount,x-systemd.device-timeout=10,timeo=14,x-systemd.idle-timeout=1min 0 0
diskstation:/volume1/web    /data/nfs/web   nfs auto,x-systemd.automount,x-systemd.device-timeout=10,timeo=14,x-systemd.idle-timeout=1min 0 0
```
[/details]

[details="10-net-services"]
```
/data/private/scripts >>> cat 10-net-services                                                                                                                   
#!/bin/bash
echo "Using sudo ..."
echo "Start and enable nfs-client.target"
sudo systemctl enable --now nfs-client.target
sudo systemctl enable --now NetworkManager-wait-online.service
```
[/details]


[details="20-create-data-structure"]
```
/data/private/scripts >>> cat 20-create-data-structure                                                                                                          
#!/bin/bash
datadir="/data/"
echo "Creating Data mount structure"
sudo mkdir ${datadir}
sudo mkdir ${datadir}/nfs
sudo mkdir ${datadir}/build
sudo mkdir ${datadir}/private
sudo mkdir ${datadir}/projects
sudo mkdir ${datadir}/virtualbox
sudo chown $USER ${datadir}/nfs \
                 ${datadir}/build \
                 ${datadir}/private \
                 ${datadir}/projects \
                 ${datadir}/virtualbox
mkdir ${datadir}/nfs/data \
      ${datadir}/nfs/music \
      ${datadir}/nfs/photo \
      ${datadir}/nfs/video \
      ${datadir}/nfs/web
```
[/details]


[details="30-mod-fstab"]
```
/data/private/scripts >>> cat 30-mod-fstab                                                                                                                      
#!/bin/bash
echo "Modifying a copy of /etc/fstab..."
workdir=${PWD}
protected=$(grep 'PROTECT' /etc/fstab)
if ! [[ -z $protected ]]; then
	echo ""
	echo "Already modified! exit ..."
	exit
fi
cp /etc/fstab ${workdir}/fstab.org
echo "# PROTECTION against double merge" >> ${workdir}/fstab.org
echo "Merging 'fstab.txt' with 'fstab.org'"
cat ${workdir}/fstab.txt >> ${workdir}/fstab.org
echo "Using sudo: Replacing /etc/fstab..."
sudo mv ${workdir}/fstab.org /etc/fstab
sudo chown root:root /etc/fstab
echo "Using sudo: Mounting /etc/fstab"
sudo mount -a
echo "Done!"
```
[/details]


[details="40-link-data"]
```
/data/private/scripts >>> cat 40-link-data                                                                                                                      
#!/bin/bash
cd ~

#linking root data folder
ln -s /data ~/Data

echo "Linking Documents folder..."
rm -rf Documents
ln -sf /data/private/.home/Documents

echo "Linking Downloads folder..."
rm -rf Downloads
ln -sf /data/private/.home/Downloads

echo "Linking Music folder..."
rm -rf Music
ln -sf /data/private/.home/Music

echo "Linking Pictures folder..."
rm -rf Pictures
ln -sf /data/private/.home/Pictures

echo "Linking Manjaro tools config folder..."
ln -sf /data/private/manjaro-tools-config ~/.config/manjaro-tools

echo "Linking Thunderbird data folder..."
ln -sf /data/private/.home/.thunderbird

echo "Linking Virtual Environments folder..."
ln -sf /data/projects/.virtualenvs

echo "Linking VirtualBox VMs..."
ln -sf /data/virtualbox/VirtualBox\ VMs

echo "Copying Git config..."
ln -sf /data/private/.home/.gitconfig

echo "Linking makepkg config..."
ln -sf /data/private/.home/.makepkg.conf

echo "Linking Transifex config..."
ln -sf /data/private/.home/.transifexrc

echo "Linking netrc config..."
ln -sf /data/private/.home/.netrc ~/
chown fh:fh ~/.netrc
chmod u+rwx,go-rwx ~/.netrc

echo "Copying SSH config folder..."
cp -r /data/private/.home/.ssh ~/.ssh
chown fh:fh ~/.ssh -R
chmod u+rwx,go-rwx ~/.ssh -R

echo "Linking thunderbird folder..."
ln -sf /data/private/.home/.thunderbird

echo "Linking GPG keyring"
rm -f ~/.gnupg
ln -sf /data/private/.home/.gnupg
chown fh:fh ~/.gnupg -R
chmod u+rwx,go-rwx ~/.gnupg -R

echo "Done!"

```
[/details]
