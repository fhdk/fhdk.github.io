---
published: true
date: '01-10-2019 00:00'
publish_date: '01-10-2019 00:00'
title: 'USB partition script'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

**Difficulty: ★★☆☆☆**

## Description

The scripts partitions and formats any USB device using **exFAT** filesystem. To avoid accidental format of non removable devices a couple of safeguards exist.

The script will check if the supplied device is valid then it checks if the device is in fact a removable device. If any of these checks fail the script aborts.

The script will ask for user confirmation twice and default to abort on <kbd>Enter</kbd>

## Use case
Re-purpose the USB stick if you have flashed an ISO to it.

Why? Because writing an ISO using `dd` writes an ISO9660 filesystem to a small part of the USB - the rest is inaccessible.

## Instructions
Create the folder **~/.local/bin**
```
mkdir -p ~/.local/bin
```

Create a new file e.g. **format-usb.sh**
```
touch ~/.local/bin/format-usb.sh
```
Make the file executable
```
chmod +x ~/.local/bin/format-usb.sh
```
Open the file with your favorite editor e.g. kate
```
kate ~/.local/bin/format-usb.sh
```
Copy below content into the file and save it.

**TIP**: Use the copy to clipboard box in the upper right corner of below text box

```

#!/usr/bin/env bash
#
#    Utility ccript to repurpose a removable device using exfat filesystem
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
#    along with this program.  If not, see <https://www.gnu.org/licenses/>.#
#
# @linux-aarhus - root.nix.dk
#

# format removable USB device to exfat with name 'USB STICK'
LABEL='USB STICK'

SCRIPTNAME=$(basename "$0")
VERSION="0.2"

# don't run as root
if [[ $(whoami) == "root" ]] ; then
    echo "Please run as user. Script will ask for root later."
    exit 1
fi

# ensure a device is given
if [[ -z "$1" ]]; then
    echo "No device specified ..."
    echo "Usage: ${SCRIPTNAME} /dev/sdy"
    exit
fi

# get the last part of device path
device="$(echo $1 | cut -d'/' -f3)"
echo "Checking device on => $device"

# create list of available devices
devices="$(lsblk -o NAME | egrep '^sd')"

# check if device is valid - abort if not
if ! [[ $devices =~ (^|[[:space:]])$device($|[[:space:]]) ]]; then
    echo "$1 not found"
    echo "Aborting"
    exit 1
fi

# check if device is removable - abort if not
[[ $(echo $(lsblk -no RM "$1" | head -n 1)) = '0' ]] && \
    echo "Non removable device detected!" && \
    echo "Aborting" && \
    exit 1

# Ask for confirmation
read -r -p "Confirm reformat of '$1' [y/N] " resp
if [[ "$resp" =~ ^([yY][eE][sS]|[yY])$ ]]; then

    # Repeat confirmation question
    read -r -p "Irrevocably format  '$1' [y/N] " resp2
    if [[ "$resp2" =~ ^([yY][eE][sS]|[yY])$ ]]; then

        # will do
        echo "Formatting $1 ..."

        # use gdisk to create new partition table
        # and a single Microsoft basic data partition type
        printf 'o\ny\nn\n\n\n\n0700\nw\ny\n' | sudo gdisk "$1" > /dev/null
        sudo mkexfatfs -n "${LABEL}"  "$1"1 > /dev/null

        # done
        echo "Done"
        exit
    fi
    echo "Formatting aborted"
else
    echo "Formatting aborted"
fi
```

## Usage
Remove your USB if attached and list devices

    lsblk

Insert your USB device and list the devices

    lsblk

The new device in the list is your USB device e.g. /dev/sdy - format the device using the script

    format-usb.sh /dev/sdy

## Note
What this script does can be done in numerous ways with numerous tools.

The **list contains** validation is taken from this [stackoverflow topic][1]

### Revisions
**2021-01-11 06:44:00 Europe/Copenhagen**

* Added label to format
* Cosmetic fix -> send output to /dev/null

**2020-09-02 17:54:00 Europe/Copenhagen**

* Fix typo in device check

[1]: https://stackoverflow.com/a/8063398