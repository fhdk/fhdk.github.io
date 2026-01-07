---
title: 'Set Browser Script'
taxonomy:
    category:
        - docs
---

## Setting web browser script

* How do I set the default browser on Linux - more specifically Arch based Manjaro and EndeavourOS?
* Why does by apps not respect my chosen browser?

How often have we heard such question?

I just came across an issue where my development environment did not open the expected browser.

This occurs when the mime settings is out of sync with each other. To get my system to behave I put together this tiny script.


```

#!/bin/bash
#
# Setting default browser using xdg utils
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
# 2021-11-20
# @linux-aarhus (c) root.nix.dk
# https://root.nix.dk/en/utility-scripts/set-browser-script
#

print_usage(){
    echo "Usage:"
    echo "  Supply browser desktop file as the first argument"
    echo "  e.g.   set-browser.sh firefox.desktop"
    echo ""	
}

show_available_browsers(){
    echo "Available browsers (installed)"
    echo "---"
    # funny thing with cut when splitting a path
    # one would expect the column to be number 4 but it is 5
    # as string starts with `/` and therefore first column is empty
    grep -rl 'internet' /usr/share/applications | cut -d'/' -f5
    echo "---"	
}

set_browser() {
    # set scheme handlers
    xdg-mime default "$1" x-scheme-handler/https
    xdg-mime default "$1" x-scheme-handler/http
    # set default browser
    xdg-settings set default-web-browser $1
    echo "Default browser set using $1"
}

# check if argument is supplied
if [[ "$1" == "" ]]; then
    print_usage
    show_available_browsers
    exit 1
fi

# check if desktop file
if [[ "$1" =~ ".desktop" ]]; then
    # check if the file exist
    if [[ -f "/usr/share/applications/$1"  ]]; then
    	set_browser "$1"
    else
        echo "$1 - file not found"
        show_available_browsers
        exit 1
    fi
else
    echo "$1 is not a .desktop file"
    show_available_browsers
    exit 1
fi

```