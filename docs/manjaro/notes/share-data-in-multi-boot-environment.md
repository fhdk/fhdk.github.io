---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Sharing data in multi-boot environment'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Synopsis

Some Linux users like to experiment with their system and more often than not it leads to the decision of reinstalling the system. 

For the purpose of increased usability, uptime or other reasons some like to have several systems installed 

That repeatedly creates the issue on how to handle not only the home folder but also data you would like to share between distributions - that can be configurations and/or personal files.

The purpose of this document is to create an easy and convenient way of safeguarding or sharing personal data files while retaining the home folder.

## Preparation
One comment I always hear is *you can't make folders outside the folder standards* so here is the a [link to the definition]
(http://www.pathname.com/fhs/pub/fhs-2.3.html#INTRODUCTION) and I quote

> The FHS document has a limited scope:
>     Local placement of local files is a local issue, so FHS does not attempt to usurp system administrators.

So to emphasize **Local placement of local files is a local issue**.

Your system would ideally have two disks but if that is not possible - laptops come to mind - then a custom partitioned system with three or more partitions partitions or more will suffice.

As we will be moving folders which traditionally are those taking most space like  ***Documents*, *Downloads*, *Music*, *Pictures*, *Video*** we will be keeping the user's home folder as the standard location */home/$USER*

You can easily do this without reinstalling - suffice you have enough diskspace to resize your root partition. **Don't touch your $esp partition** - it will probably make your system unbootable.

1. Efi System Partition (ESP)
2. Root with home
3. User data

If you are not reinstalling then begin with resizing either the root partition or another suitable partition.

If you choose to resize your root partition - you need reboot your system on a live media with the suitable software as a mounted partition cannot be shrinked only extended.

## Setting up the partition

### Mount point
Create a mount point in the root of the file system and set permissions.

    sudo mkdir /data/shared
    sudo chmod ugo+rws /data/shared
    
Next step is to mount the partition and modify fstab to mount it at boot. The simple and preferred method is to use the **Gnome Disk Utility** and modify the mount options so the partition mounts at boot time in the **/data/shared** folder.

### Foldernames in this document
This document will assume xdg-user-dirs english foldernames. You can of course adapt it entirely to your preferences. So lets move to creating folders.

### Created folders
Ensure your partition is mounted at the designated mountpoint before continuing.

Create the folders you want to share between systems or between users e.g. for Music, Pictures and Documents.

    mkdir /data/shared/Music /data/shared/Pictures  /data/shared/Documents

The */shared* folder is writeable by everyone so if you are maintaining a multi user system you can furter adapt the folder structure by creating a $USER folder with permissions for the user only e.g.

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

Depending in the folders you want to control you must ceate a symbolic link in your home to the new location. 

Decide which folders you want to control and create a new file in your `~/Data/scripts` folder and make it executable

    $ touch ~/Data/scripts/30-link-data
    $ chmod +x ~/Data/scripts/30-link-data

Edit the file and and paste the following content into it. Modify the script to match your system and the folders you want to control and save the file. If want bulletproof folder structure consider the [information here](https://forum.manjaro.org/t/wiki-move-your-personal-data-to-different-partition/47790/8?u=fhdk)

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
echo "Creating Data mount structure"
mkdir -p ${datadir}
```

The script is used to create your desired data structure. Add what you deem necessary.

## Conclusion

You have now created three scripts and a text file which can be used whenever you have reinstalled your system.

Just run the scripts in sequence 10- 20- 30- and your data is mounted, linked and available - all within minutes.

The scripts folder can be copied to a USB stick and run from it or be kept in the data partition and run from there.
