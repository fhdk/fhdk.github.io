---
title: 'Battery Charge Notifier'
taxonomy:
    category:
        - docs
---

A recent [topic][1] caused me to add the following script to my toolbox.

The script is configurable and uses an interval and lower/upper threshold for battery.

The script utilizes libnotify to send a system message when outside the limits - thus ensuring you don't forget to plug or unplug your laptop charger.

## Script

Create the local bin folder

```
mkdir ~/.local/bin
```

Create a new file and make it executable

```
touch ~/.local/bin/charge-notify.sh && chmod +x ~/.local/bin/charge-notify.sh
```
Edit the file with kate or another editor

```
xdg-open ~/.local/bin/charge-notify.sh
```

Paste below content and save

```

#! /bin/bash
#
# Script to notify when battery is outside levels - time to plug charger.
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
# Source: https://linoxide.com/remind-unplug-charging-laptop-arch-linux
#
# https://forum.manjaro.org/t/set-up-an-alarm-notification-when-battery-reaches-full-charge/92331
#
# @linux-aarhus - root.nix.dk
#
# 2021-11-27
# 2021-11-28 revised - checks not updating
#                    - fix variable check on all levels
#

set -eu

# dependency check
if ! [[ "$(which notify-send)" =~ (notify-send) ]]; then
	echo "Please install libnotify to use this script.\n"
	echo "   sudo pacman -S libnotify"
	exit 1
fi
if ! [[ "$(which acpi)" =~ (acpi) ]]; then
	echo "Please install acpi to use this script.\n"
	echo "   sudo pacman -S acpi"
	exit 1
fi

# check interval (seconds)
INTERVAL=30

# example battery levels
# these leves are not based on scientific evidence
# you are required to adjust as appropriate for your device
MIN_BAT=10     # low water mark
MAX_BAT=60     # high water mark

get_plugged_state(){
	echo $(cat /sys/bus/acpi/drivers/battery/*/power_supply/BAT?/status)
}

get_bat_percent(){
    echo $(acpi|grep -Po "[0-9]+(?=%)")
}

# primary loop
while true ; do

	if [ $(get_bat_percent) -le ${MIN_BAT} ]; then # Battery under low limit
 		if [[ $(get_plugged_state) = "Discharging" ]]; then # plugged
 		    notify-send "Battery below ${MIN_BAT}. Time to plug adapter"
 		fi
    fi
	if [ $(get_bat_percent) -ge ${MAX_BAT} ]; then # Battery over high limit
 		if [[ $(get_plugged_state) = "Charging" ]]; then # plugged
 			notify-send "Battery above ${MAX_BAT}. Time to unplug adapter"
 		fi
	fi
	sleep ${INTERVAL} # Repeat every $INTERVAL seconds
done

```

## Usage
The battery levels are example levels. Edit the battery levels according to your system and preference.

If in doubt please see below comments and do your own research.

Launch the script using the systems autorun feature - each system does it differently :slight_smile:

* https://forum.manjaro.org/t/root-tip-charger-notifier-battery-state/92337

[1]: https://forum.manjaro.org/t/set-up-an-alarm-notification-when-battery-reaches-full-charge/92331