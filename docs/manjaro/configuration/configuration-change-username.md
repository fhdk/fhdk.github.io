---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Change username'
taxonomy:
    category:
        - docs
metadata:
    Author: linux-aarhus
---

## Change username
---
Username should be chosen with care for while it is possible to change it - it may have implications beyond recognition.

## Implications to consider
---
* Configuration file may no longer work as expected.
* Scripts and service units relying on the old username.
* Symlinks may point to invalid destinations.

If you still think you are up for the change - please read on.

## The GNU way
---

By reading the man pages you find the GNU system has thought of it.
```
    man usermod
    man groupmod
```
### Either

* Reboot and don't login.
* Open a TTY pressing <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F4</kbd>

### Or

* Reboot and edit the grub entry.
* Press <kbd>e</kbd> on the selected grub entry and add the number <kbd>3</kbd> to the kernel command line and press <kbd>F10</kbd> to continue boot.

## Login as **root**!
---

You **cannot** login as the user you want to change - you **must** use root.

Manjaro usually creates a user group matching the username and assigns the user to the specific group. This makes it necessary on Manjaro to rename both the login and group.

When you are logged in as **root** first rename the user and rename the user's home

```bash
# usermod --login $NEWLOGIN  --move-home --home /home/$NEWLOGIN $OLDLOGIN
```

Then rename the user's group
```
# groupmod --new-name $NEWGROUP $OLDGROUP
```

NOTE: This can also be done from a live ISO using chroot.

## Remedy implications
Contributed by [@freggel.doe][1] at Manjaro Forum

Some symlinks in that new home-directory might be broken after, because their target contain the full path with the old username. Wine and it's default `$HOME/.wine` -prefix comes to mind.

To find dead symlinks:

```
find /home/$NEWLOGIN -xtype l
```

Some config-files contain full paths as well (desktop background image for example). To find files with "old" username:

```
grep -rn "/home/$OLDLOGIN/"
```

[1]: https://forum.manjaro.org/u/freggel.doe
