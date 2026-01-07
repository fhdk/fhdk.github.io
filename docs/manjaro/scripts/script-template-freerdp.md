---
title: 'Get LAN IP address'
taxonomy:
    category:
        - docs
---

This is a utility script to display your local network address.

Intended use case is anywhere you need an unambigious reference to your current LAN IPv4 whether this is another script or just to make sure.

Add a file to your `~/.local/bin` folder, name it `check-network`, make it executable 
```

mkdir -p  ~/.local/bin
touch ~/.local/bin/check-network
chmod +x ~/.local/bin/check-network
```

Edit the file and paste below content - be aware that the ttf_icons() does not display correct.

The function looks like this in Sublime Text. 
The font used is the package `ttf-font-icons` available in the Manjaro repo.

```

#! /bin/bash
#
# Script for displaying local network address in polybar, tint2, conky etc.
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
# for icons install the package ttf-font-icons from the repo
#

ttf_icons(){
    # set icons if font is available
    if [ -f '/usr/share/fonts/TTF/icons.ttf' ]; then
      wlan=''
      lan=''
      offline=''
    fi
}

lanip() {
    # parse output from ip a show $nic
    echo $(ip a show $1 | grep 'inet ' | head -n 4 | awk '{print $2}' | cut -d'/' -f1)
}

ttf_icons
# find active interface
nic=$(ip a | grep ' state UP' | cut -d' ' -f2 | cut -d':' -f1)
if [[ ${nic} != "" ]]; then
    # print IP address
    if [[ ${nic} == e* ]]; then
        echo ${lan} $(lanip ${nic})
    else
        echo ${wlan} $(lanip ${nic})
    fi
else
    # print offline
    echo ${offline} n/a
fi

```

This has been posted on [Manjaro Forum](https://forum.manjaro.org/t/root-tip-utility-script-get-lan-ip-address/100603) too.