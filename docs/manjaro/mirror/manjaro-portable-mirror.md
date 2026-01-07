---
title: 'Portable Manjaro Mirror'
date: '10:21 10-05-2024'
taxonomy:
    category:
        - docs
permissions:
    inherit: true
    authors:
        - linux-aarhus
---

## Why a portable mirror

There could be any number of reasons to use a portable mirror - some common use cases

- You have work to do and you don't want to destablize your perfect system but you need a specific package

- Your internet connection for syncing (installing) is slow or metered

- The Nvidia drivers work just the way they should

- To be able to travel and you ind you desperately need an arbitrary package

- The desktop is mostly bugfree -> Gnome or Plasma  :eyes: 

## Description
This topic is intended to provide a hands-on - proof of concept - on how to honor any of the above expectations

Using a portable mirror is no different than using an online mirror - except the fact that you become your own mirror proovider.

Technically - with out too much hazzle - you can expand the availability to your local network - but for now let us stay with the **portable mirror** abstraction.

## Requirements

1. Initial requirement of a reasonable fast and reliable internet connection. Could be your campus, a public service or a friend's unlimited internet connection.

2. Initial data transfer of ~100GB data - the initial mirror sync.

3. A storage device capable of holding this initial transfer plus the overhead of temporary files making a device with at least 128GB free space.

4. A lot of patience (see the Patience section in the sidebar)

:warning: Do not cheap out on the device - use a branded device with good reputation of reliability and resilience to accidents like drops or moisture.

A normal Manjaro stable branch sees fewer sync than testing and unstable, so scheduling a monthly sync of your portable mirror could be sufficient.

But as your own mirror provider - you may choose any other schedule that fits into your workflow.

## Preparation
Create a Linux partition with (as miniminum) 128GB space and format it with ext4 and the label **PortableMirror** - **do not use btrfs** even if you prefer - it is not suited for this purpose.

To avoid accidental change of your PortableMirror device you should only change the device using root context.

This means you run the rsync script using sudo as this will ensure some integrity against accidental changes.

This also means we create the initial script using root context.

Mount the partition using the label **PortableMirror** on the designated temporary mountpoint **/mnt**.

```text
sudo mount /dev/disk/by-label/PortableMirror /mnt
```

Create a new file on the new partition - call it **rsync-manjaro.sh**. I am using **micro** as my terminal editor - replace micro with your preferred editor capable of obtaining root permissions for saving the file.

```text
micro /mnt/rsync-manjaro.sh
```

Paste the following and save the file

```text
#!/usr/bin/env bash

# This must be a relative path starting with the scripts location
# To accomodate for various ways of executing the script I have used
# this snippet to ensure we always know the exact path of the script
# https://www.baeldung.com/linux/bash-get-location-within-script
SCRIPT_PATH="${BASH_SOURCE}"
while [ -L "${SCRIPT_PATH}" ]; do
  SCRIPT_DIR="$(cd -P "$(dirname "${SCRIPT_PATH}")" >/dev/null 2>&1 && pwd)"
  SCRIPT_PATH="$(readlink "${SCRIPT_PATH}")"
  [[ ${SCRIPT_PATH} != /* ]] && SCRIPT_PATH="${SCRIPT_DIR}/${SCRIPT_PATH}"
done
SCRIPT_PATH="$(readlink -f "${SCRIPT_PATH}")"
SCRIPT_DIR="$(cd -P "$(dirname -- "${SCRIPT_PATH}")" >/dev/null 2>&1 && pwd)"

# this will be the folder holding the portable mirror
TARGET="${SCRIPT_DIR}/PortableMirror"

# working variables
TMP="${SCRIPT_DIR}/tmp"

# lockfile will prevent multiple instances of the script process
# if the script is terminated mid execution
# you may need to manually remove the lockfile
LOCK="${TMP}/rsync-manjaro.lock"

# this is the mirror to sync from
# you æmay need to evaluate various mirrors to
# find one that supports checksum
SOURCE="rsync://ftp.halifax.rwth-aachen.de/manjaro/"
# assume checksum
CHECKSUM="-c"
# verify if checksum can be used
check=$(rsync --checksum --dry-run ${SOURCE})

[[ ${check} =~ "disabled" ]] && unset CHECKSUM

# create the targets
[ ! -d "${TARGET}" ] && mkdir -p "${TARGET}"
[ ! -d "${TMP}" ] && mkdir -p "${TMP}"

# exit if lockfile exist
exec 9>"${LOCK}"
flock -n 9 || exit

# if running from a timer or a cron job silence the output
if ! stty &>/dev/null; then
    QUIET="-q"
fi

if [[ -z $QUIET ]]; then
    echo "--== SYNC PORTABLE MIRROR ==--"
    echo "${SOURCE}"
    echo "please wait ... "
fi

##################################################
###
### This rsync command creates a copy of stable repo
### where symlinks are converted to regular files
### Expect the size to be around 100GB
### It is possible to reduce the size further
###  by excluding signatures
###  this requires to execute the update as
###  SigLevel=TrustAll sudo pacman -Syu
###
##################################################
rsync -rtLv --safe-links \
    --delete-after --progress -h \
    --timeout=600 --contimeout=300 -p \
    --delay-updates --no-motd \
    --exclude="arm*" \
    --exclude="pool" \
    --exclude="testing" \
    --exclude="unstable" \
    --exclude="stable/kde-unstable" \
    ${QUIET} ${CHECKSUM} \
    ${SOURCE} \
    "${TARGET}"

rm -f ${LOCK}

if [[ -z $QUIET ]]; then
    echo "${SOURCE}"
    echo "Sync complete"
fi
```

If you have the internet connection and the time - run the script

```text
sudo bash rsync-manjaro.sh
```

Whenever you have the option - once or twice a month - attach the device - mount it - open a terminal at the mount path - and execute above command to sync the portable mirror.

## Patience
For documentation purpose I timed the initial sync using an ordinary folder on my system's NVMe.

The number is for system connected to a Cat6 LAN, connected to the internet with 1Gbit fiber optic connection in Denmark.. You need to adjust your expectation according to your bandwidth.

Besides the download time you need to factor in how fast data can be written to you selected storage device - which is why you should not not cheap out on the device.

```
 $ time ./rsync-manjaro.sh
[...]
sent 568,87K bytes  received 92,38G bytes  14,12M bytes/sec
total size is 92,35G  speedup is 1,00

real    109m4,842s
user    0m20,322s
sys     2m8,504s
```

## Using the portable mirror

Your portable mirror goes into the custom mirrors catagory and according to the manpage you can use a custom mirror as a file based miror or server based mirror.

If you want to make it available in an air-gapped network (understood as no internet connection) then use a http based mirror.

If you want to use on a single air-gapped system  (with no network at all) the file based mirror is also an option.

In either case you need to mount the partition - and because we labelled the disk you do not need to know the device path - just remember the label. You could even attach a sticker to the device with the label name - easy recognizable on the desk.

### Mirror configuration
The core application **pacman-mirrors** allows you to specify a given mirror using command line.

First mount the device

```text
sudo mount /dev/disk/by-label/PortableMirror /mnt
```

For a file based mirror

```text
sudo pacman-mirrors --api --url file:///mnt/PortableMirror
```

The process for a http mirror is slightly different as you must first start a http service. The system Python contains a simple http service you can utilize for this purpose.

```text
python -m http.server -d /mnt/PortableMirror 8080 &
sudo pacman-mirrors --api --url http://localhost:8080
```

### Static configuration

If are going all-in on being your own mirror provider - you may choose to make your mirror configuration static. Edit **pacman-mirrors.conf** as root and uncomment the **# Static =** line and add your mirrors url and save the file.

### When ready

```
sudo pacman -Syu
```

## Conclusion

While setting up the portable mirror initially requires some work and patience to sync the stable branch repo, you are now able to install any application offline.

Remember this:

- pamac provides a timer which will remove your portable mirror
- pamac provides a notification when new packages can be synced

If you are going to rely on your portable mirror - it is advised to disable the timer and notification.

Instead of syncing your computer - sync your portable mirror - then sync your system using the portable mirror.

Posted at: [Create and use a portable Manjaro mirror](https://forum.manjaro.org/t/root-tip-how-to-create-and-use-a-portable-manjaro-mirror/161252)