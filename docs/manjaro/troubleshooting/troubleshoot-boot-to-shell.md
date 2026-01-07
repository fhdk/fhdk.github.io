---
title: 'kernel init'
taxonomy:
    category:
        - docs
---

- Boot your system to the grub menu. <kbd>ESC</kbd>
- Edit the menu entry with <kbd>E</kbd>
- Add at the line starting with `linux` this: 
```
init=/bin/bash
```
- Type the function key to boot.
- Now at the bash session remount the root filesystem writeable:
```
mount -n -o remount,rw /
```
Now you have root .... careful now

Surely dangerous, remember though - you must have physical access to the machine to do things like this.