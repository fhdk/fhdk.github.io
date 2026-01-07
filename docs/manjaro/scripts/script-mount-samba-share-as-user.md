---
title: 'Mount samba share as user'
taxonomy:
    category:
        - docs
---

**Difficulty: ★☆☆☆☆**

## Utility to mount samba share as user

While gvfs is an acronym for Gnome Virtual File System it's usage is not depending on Gnome. It comes with very few dependencies and even Manjaro Plasma has the gvfs package installed and as such this will work on any Linux.

The Linux mount utility does not allow for a user to mount a samba share and this results in Manjaro users setting up fstab to automount the share. Such automounts may cease working due to upstream changes e.g. when the SMB1 protocol was renamed to NT1 thus causing old shares to stop working.

This script relies on the packages `gvfs-smb` which will pull the necessary `smb-client` dependency. The following command will do the trick. If you want the dependencies to be explicitly installed add `gvfs` and `smbclient` to the command. The `--needed` argument will skip syncing packages already present.

    sudo pacman -Syu gvfs-smb --needed

Create a new script in `~/.local/bin` and name it `gio-mount-myshare.sh`.

    mkdir -p ~/.local/bin
    touch ~/.local/bin/gio-mount-myshare.sh

Paste below content and modify the variables to use your samba server and share name.

The symlinks folder defaults to `~/SMBLinks` and this can of course be changed as you see fit.

Running the script will attempt mounting the share. If credentials are needed you will be challenged. Enter the correct credentials to continue. To unmount a mounted share run with `-u` argument.

To avoid entering credentials at runtime you can fill in the credentials inside the script - they will then be fed to the mount option using a file in a hidden configuration folder _.credentials_.

```

#! /bin/bash
#
# Script for mounting and unmounting a configured samba share
#  Customized the variables as needed
#  - Symlinks are located in designated folder (default: $HOME/SMBLinks)
#  - Symlink can be named  (default: $SHARENAME)
#  - Using `-u` argument will unmount and remove the symlink
#
#    This program is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License
#    along with this program.  If not, see <https://www.gnu.org/licenses/>.
#
# @linux-aarhus - root.nix.dk
#

# MODIFY THESE VARIABLES
# your samba server's hostname or IP address
HOST="my-server"

# the share name on the server
SHARENAME="my-share"

# symlink name
LINKNAME="$SHARENAME"

# symlink is placed in this folder
SYMLINKS="$HOME/SMBLinks"

# credentials
USERNAME=
WORKGROUP=
PASSWD=

# /END MODIFY

SCRIPTNAME=$(basename "$0")
VERSION="0.2"

# check argument $1
if [[ "$1" == "-u" ]]; then
    # remove symlink
    rm -f "$SYMLINKS/$LINKNAME"
    # unmount share
    gio mount -u "smb://$HOST/$SHARENAME"
    exit
elif [[ $1 != "" ]]; then
    echo ":: $SCRIPTNAME v$VERSION"
    echo "==> invalid argument: $1"
    echo "Usage: "
    echo "  mount SMB : $SCRIPTNAME"
    echo "  umount SMB: $SCRIPTNAME -u"
    exit 1
fi

# Create credentials folder
mkdir -p $HOME/.credentials
chmod 700 $HOME/.credentials

# ----------------------------------------------------------------
# mount command
if ! [[ -z "${USERNAME}" ]]; then
    # create credentials file
    fname="$HOME/.credentials/$USERNAME-$HOST-$SHARENAME"
    printf ${USERNAME}'\n'${WORKGROUP}'\n'${PASSWD}'\n' > $fname
    chmod 600 $fname
    # mount and feed the credentials to the mount command
    gio mount "smb://$HOST/$SHARENAME" < $fname
else
    # mount (if credentials are required you will be prompted
    gio mount "smb://$HOST/$SHARENAME"
fi
# ----------------------------------------------------------------

# easy reference to gio mount point
ENDPOINT=/run/user/$UID/gvfs/''smb-share:server=$HOST,share=$SHARENAME''

# crete the subfolder
if ! [ -d "$SYMLINKS" ]; then
    mkdir -p "$SYMLINKS"
fi

# use --force argument to suppress warning if link exist
ln -sf "$ENDPOINT" "$SYMLINKS/$LINKNAME"

```

You can complement this with a systemd user service to automate things even more. 

Create the folder
```
mkdir -p ~/.local/systemd/user
```
Create the service file
```
touch ~/.local/systemd/user/gio-mount-myshare.service
```
Edit the service file with your favorite text editor and paste below content
```
[Unit]
Description=GIO mount smb share-name

[Service]
Type=oneshot
ExecStart=/home/%u/.local/bin/gio-mount-myshare.sh
ExecStop=/home/%u/.local/bin/gio-mount-myshare.sh -u
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```
Enable and start the service

    systemctl --user enable gio-mount-myshare.service

To simplify maintenance you can move the script to the service folder and change the ExecStart and ExecStop paths in the service file to _/home/%u/.local/systemd/user/_.

Posted at [Manjaro Forum [root tip] [How To] [Utility script] GIO mount samba share ][1]

[1]: https://forum.manjaro.org/t/root-tip-how-to-utility-script-gio-mount-samba-share/100723