---
title: 'Arch ARM image WiFi hack'
date: '06:23 14-10-2022'
taxonomy:
    category:
        - docs
---

**Difficulty: ★★☆☆☆**

## Preconfigure ARM WiFi

The utility script preconfigures the sd-card to connect to WiFi at first boot.

After you have written the installer image to your sd-card, run this script to configure a connection to your wifi

Save the script on your system as e.g. **wifi-hack.sh**.

The script takes 3 mandatory arguments but you can opt to create a static configuration add 3 more arguments.

```
<blockdev> <ssid> <passphrase> [ <cidr> <gateway> <dns> ]
```

With 3 arguments the wlan is setup using DHCP - with 6 arguments the wlan is configured for static addressing.

Sample command line - adapt the configuration to your use case and ensure no conflicts using the IP address
```
sudo bash wifi-hack.sh /dev/mmcblk0 my-ssid my-passphrase 192.168.1.10/24 192.168.1.1 192.168.1.1
```

```
#!/usr/bin/env bash
#
# Script to preconfigure ARM installer for WiFi connection
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
# root.nix.dk
#
set -e
scriptname=$0
tmproot="/tmp/root"
usage(){
    # <blockdev> <ssid> <passphrase> [ <cidr> <gateway> <dns> ]
	echo "Usage: sudo ${scriptname} <blockdev> <ssid> <passphrase> [ <cidr> <gateway> <dns> ]"
	exit 1	
}

# run as root
if ! [[ $(whoami) == "root" ]] ; then
	echo "Run script as root using su or sudo"
    exit 1
fi

# check if arg1 is block device
if ! [[ -b $"$1" ]]; then
    echo "'$1' is not a block devicce"
    exit 1
fi

# mount partition
echo Mount sd-card root partition
mkdir -p "${tmproot}"
mount "$1"p2 "${tmproot}"

# create wlan0 network
# if we have 3 arguments - use dhcp
if [[ $# -eq 3 ]] ; then
cat << EOF >> "${tmproot}/etc/systemd/network/wlan0.network"
[Match]
Name=wlan0
[Network]
DHCP=yes
EOF
# if we have 6 arguments - static ip
elif [[ $# -eq 6 ]]; then
cat << EOF >> "${tmproot}/etc/systemd/network/wlan0.network"
[Match]
Name=wlan0
[Network]
Address=$4
Gateway=$5
DNS=$6
EOF
# argument count must be either 3 or 6
else
	usage
fi

# create wpa_supplicant.conf
wpa_passphrase "$2" "$3" > "${tmproot}/etc/wpa_supplicant/wpa_supplicant-wlan0.conf"

# create symlink for wlan0 service
ln -sf "/usr/lib/systemd/system/wpa_supplicant0.service" \
      "${tmproot}/etc/systemd/system/multi-user.target.wants/wpa_supplicant@wlan0.service"

# unmount and clean up
echo Unmounting
umount ${tmproot}
echo Cleaning up
rmdir ${tmproot}
echo Done
```

Posted at
* [https://forum.manjaro.org/t/root-tip-how-to-utility-script-preconfigure-arm-installer-for-wifi/124240](https://forum.manjaro.org/t/root-tip-how-to-utility-script-preconfigure-arm-installer-for-wifi/124240)
* [https://forum.endeavouros.com/t/hacking-arm-image-for-wifi-config/32888](https://forum.endeavouros.com/t/hacking-arm-image-for-wifi-config/32888)