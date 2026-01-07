---
title: 'Changing LUKS passphrase'
date: '09:16 03-03-2024'
taxonomy:
    category:
        - docs
---

## Locate key slot
Assuming the partition holding the luks volume is /dev/sda2

As you may have more than one slot active - locate the one for the passphrase
```
cryptsetup open --type=luks --test-passphrase -vv /dev/sda
No usable token is available.
Enter passphrase for /dev/sda2:
Key slot 1 unlocked.
Command successful.
```
## Add new key slot
In this case the slot is number 1 so add a new key
```
cryptsetup luksAddKey /dev/sda2
```
## Wipe previous key slot
Then wipe the slot for the previous phrase (slot returned from the check) - enter the new passphrase when challenged
```
cryptsetup luksKillSlot /dev/sda2 1
```