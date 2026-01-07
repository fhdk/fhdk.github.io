---
title: 'Check AUR buildscript changes'
date: '06:37 20-12-2022'
taxonomy:
    category:
        - docs
publish_date: '15-04-2023 06:45'
published: true
---

!! ## AUR != Manjaro stable branch

Test if you are AUR ready by running below snippet in a terminal

```text
if [[ $(pacman-mirrors -G) == 'stable' ]]; then echo 'AUR is a no-go'; else echo 'OK - go ahead'; fi
```

## The Arch way

Every AUR build script page has a notification subscription -> in upper right box. 

This is a simple as it gets - when the script is updated you get a mail notification.

## But Pamac can do that?

And you are correct - it can.

Even so it has proven countless times this is a bad idea.

1. An otherwise successful sync breaks due invalid AUR database
2. You run into issues when a package and dependencies is demoted from repo to AUR
3. Duplicate package names - where version in AUR is higher than official repo. 
   [[Example 1][1]] [[Example 2][2]]

That just a couple of stumbling points the last couple of months.

So do yourself a favour and revert Pamac -> Preferences -> Third Party -> Check for updates to the default - which is disabled.

Another fairly simple solutions is to run a script after login

## The check-aur.sh script

The script requires **yay**  to be installed

```text
sudo pacman -Syu yay
```

!!! Due to a - @2023-05-09 -  fair amount of [open bugs][yaybugs] with **yay** you may choose to use **paru** instead.

Create a script - **check-aur.sh** - place it in **~/.local/bin** and make it executable with content

### Notify script

```text
#!/usr/bin/env bash
#
# AUR buildscript update checker
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
# 2022-12-20
# @linux-aarhus (c) root.nix.dk
# 2023-05-09: added not on yay bugs

# due to a fair amount of open bugs for yay https://github.com/Jguer/yay/issues
# it may be due dilligence to change to paru
# paru supports the same arguments as yay
# simply change yay to paru in the script

AurHelper="yay"

#check if yay is available
if ! command -v ${AurHelper} > /dev/null; then 
    echo ":: ${AurHelper} not found... please install"
    exit 1
fi

#check if libnotify is available
if ! [[ "$(which notify-send)" =~ (notify-send) ]]; then
	echo ":: libnotify not found... (sudo pacman -S libnotify"
	exit 1
fi

aur_changed=$(${AurHelper} -Quaq)

if ! [[ -z ${aur_changed} ]]; then
    if ! stty &>/dev/null; then
        notify-send -u normal "Notification of AUR changes"  "$aur_changed"
        exit
    fi
    printf "${aur_changed}"
fi
```

Make the script executable

```text
chmod +x ~/.local/bin/check-aur.sh
```

The script will do

- check if yay is installed - if not exit
- execute `{paru|yay} -Quaq` and store the response in a variable
- if there is any content in the response 
  - if run by a service use the system notification daemon to pop a notification
  - if run from command line output to console

## autorun after login

To run it one time after login create a user service file

First create the service folder structure

```text
mkdir -p ~/.config/systemd/user
```

Then create a file **~/.config/systemd/user/check-aur.service** - paste the following content

```text
[Unit]
Description=Check for changes to AUR my buildscrpts
# wait for network
After=syslog.target network.target default.target

[Service]
Type=simple
ExecStartPre=/bin/sleep 30
ExecStart=/home/%u/.local/bin/check-aur.sh

[Install]
WantedBy=default.target
```

Start the user service - do not use sudo

```text
systemctl --user enable check-aur.service
```

After login the service will activate and do the following

- wait 30 seconds
- run the check-aur.sh script
- execute the script and exit

## oh-no no updates - what shall I make of my time

**If you are not satisfied with the popup shortly after login create a timer**

<big>Do not flood AUR with useless traffic</big>

Create a complementary timer **~/.config/systemd/user/check-aur.timer** with content - example is 4 hours

```text
[Unit]
Description=Schedule AUR change checks

[Timer]
OnStartupSec=10m
OnUnitActiveSec=4h

[Install]
WantedBy=timers.target

```
The timer will do this

* wait until 10m after boot
* activate the check-aur.service
* re-activate the service every 4 hours while system is up

When using a timer - you need to disable the service - otherwise the timer refuse to activate it.

```text
systemctl --user disable --now check-aur.service
```

Then activate the timer

```text
systemctl --user enable --now check-aur.timer
```

## When you are notified

When you use this notifier and you are notified - simply use your favorite AUR helper to rebuild the packages.

Pamac works from command line too

```text
pamac build <scriptname>
```

Rebuild all from the command line

```text
pamac build $(check-aur.sh)
```

## Rewritten notify script with IgnoredPkgs

This example is from my system. I am using the **dotnet-core-bin** custom script to install dotnet 6 - because the the project I work on must have long term support.

My issue is that the above script keeps informing me the package has been updated to dotnet 7 - which I - for the time being - do not need.

So I rewrote the script be able to handle an array of ignored packages. One added benefit was the aquired bash array knowledge - never too old to learn - right?

!!! Due to a - @2023-05-09 - fair amount of [open bugs][yaybugs] with yay I have added support for paru which - when available - will be preferred over yay.

```text
#!/usr/bin/env bash
#
# AUR buildscript update checker - optional ignore specific packages
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
# 2023-04-15
# @linux-aarhus (c) root.nix.dk
# Revision log
# 2023-05-09: Added paru as supported helper - paru will be preferred over yay 
#           : Rewrite comparison to arrays and fix missing content with libnotify
# 2023-05-10: shellcheck validated
#
#
# Ignore specific packages
# e.g. IgnorePkgs=('pkg1' 'pkg2')
IgnorePkgs=('dotnet-host-bin' 'dotnet-targeting-pack-bin'
           'aspnet-targeting-pack-bin' 'aspnet-runtime-bin'
           'dotnet-runtime-bin' 'dotnet-sdk-bin'
           'netstandard-targeting-pack-bin')

# to skip search set AurHelper to yay or paru
AurHelper="paru"

if [[ -z ${AurHelper} ]]; then
    #check if paru is available
    if command -v paru > /dev/null; then 
        AurHelper="paru"
    fi
    
    if [[ -z ${AurHelper} ]]; then
        #check if yay is available
        if command -v yay > /dev/null; then 
            AurHelper="yay"
        fi
    fi

    if [[ -z ${AurHelper} ]]; then
        echo "=> No AurHelper found"
        echo "=> supported helpers:"
        echo "  --> yay:  sudo pacman -S yay"
        echo "  --> paru: pamac build paru-bin"
        exit 1
   fi
fi

#check if libnotify is available
if ! [[ "$(which notify-send)" =~ (notify-send) ]]; then
        echo ":: libnotify not found... (sudo pacman -S libnotify)"
        exit 1
fi

# get changed scripts
aur=$(${AurHelper} -Quaq)

# setup arrays
IFS=$'\n' ignored=("$(sort <<<"${IgnorePkgs[*]}")"); unset IFS
IFS=$'\n' changed=("$(sort <<<"${aur[*]}")"); unset IFS

# create a filtered array
filtered=$(comm -13 <(printf "%s\n" "${ignored[@]}") <(printf "%s\n" "${changed[@]}"))

# create the notification
notification=$(for f in "${filtered[@]}"; do echo "${f}"; done)

# display notification if the filtered array is not empty
if  [[ -n "${notification}" ]]; then
    if ! stty &>/dev/null; then
        notify-send -u normal "Notification of AUR changes" "${notification}"
    else
        echo "${notification}"
    fi
fi

```

[1]: https://forum.manjaro.org/t/pamac-suggesting-aur-upgrades-to-manjaro-packages-stable-branch/131116/5
[2]: https://forum.manjaro.org/t/replace-pacman-mirrors/129896
[yaybugs]: https://github.com/Jguer/yay/issues