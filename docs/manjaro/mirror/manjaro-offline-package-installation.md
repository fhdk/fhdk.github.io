---
title: 'Manjaro - offline package installation'
date: '08:08 18-06-2023'
taxonomy:
    category:
        - docs
---

To download a package with dependencies e.g. `libreoffice-fresh`
~~~
sudo pacman -S libreoffice-fresh --cachedir ~/tmp --downloadonly  $(pacman -Si libreoffice-fresh | awk -F'[:]' '/^Depends/ {print $2}')
~~~

To update an offline installation - we need a lot more - first a disk of no less then 80G free space and a rsync script to fetch the entire Manjaro stable branch.
~~~
#!/bin/sh

#######################################################
###
### Variables to amend
### Ensure your target has enough free space
### Expect around 70-80GiB
###
TARGET="~/ManjaroRepo"
TMP="~/.cache/manjaro"
LOCK="~/.cache/rsync-manjaro.lock"

### arguments to exclude folders not needed for an offline stable repo
### exclude from sync
EXCLUDE='--exclude="arm" --exclude="pool" --exclude="testing" --exclude="unstable" --exclude="*/kde-unstable"'

### mirror rsync address 
SOURCE="rsync://mirror.easyname.at/manjaro/"

######################################################
###
### create folders if the don't exist
###
######################################################
[ ! -d "${TARGET}" ] && mkdir -p "${TARGET}"
[ ! -d "${TMP}" ] && mkdir -p "${TMP}"

# setup lock
exec 9>"${LOCK}"
flock -n 9 || exit

# if run by a systemd timer
if ! stty &>/dev/null; then
    QUIET="--quiet"
fi

##################################################
###
### This rsync command creates a copy of stable repo
### where symlinks are converted to regular files
### Expect the size to be around 70GB to 80GB
###
### rsync is an io-intensive task
### therefore some mirrors do no honor --checksum
##################################################
CHECKSUM="--checksum"
rsync --recursive --times --copy-links --verbose --safe-links \
    --delete-after --progress --perms ${CHECKSUM} \
    --timeout=600 --contimeout=300 \
    --human-readable --delay-updates --no-motd \
    ${QUIET} --temp-dir="${TMP}" \
    ${EXCLUDE} \
    ${SOURCE} \
    "${TARGET}"

rm -f ${LOCK}
~~~

~~~
#!/usr/bin/env bash
## sample script for updating a remote system with no network
## using a labeled filesystem on a portable medium

## mount using label e.g. ManjaroRepo
sudo mount /dev/disk-by-label/ManjaroRepo /mnt

## start the python http server
python -m http.server -d /mnt 8080 &

## use pacman-mirrors to point pacman to the local mirror
sudo pacman-mirrors -aU http://localhost:8080

## update the system
sudo pacman -Syyu

## optionally reboot the system
#reboot
