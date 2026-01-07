---
title: 'Smart Manjaro system update'
published: true
date: '2020-10-28 08:42'
publish_date: '2020-10-10 07:27'
taxonomy:
    category:
        - docs
---

## Manjaro updates
---
Either when you use your system or shortly after login - a message appears - telling you there is updates available. When you click the message Pamac opens with the updates pane listing 100's of packages ready to download/install on your system. Almost per instinct you click the update now - provide your password and the flow begin. Suddenly your screen goes blank - you wait - nothing happens. You press <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F4</kbd> - nothing happens - you wait - nothing happens. You hold the power button until the system powers off and press it again to power on your system - and now you are having a black screen maybe a message ....

What now? Could the situation have been  avoided?

While - of course - there is no guarantee - the following headlines may help you understand what goes on behind the scenes.

1. Know your system
2. What happens when you update
3. What happens and why when an update stalls
4. Updating the smart way
5. Conclusion

## Know your system
---
There is a few packages - which when updated most likely will require a restart. A short list of important system packages which probably requires restart

* ucode
* cryptsetup
* linux
* nvidia
* mesa
* systemd
* wayland
* xf86-video
* xorg

Another part of knowing your system is which foreign packages you have installed. Foreign packages is defined as everything not in the official repository so keeping a list of packages installed by an AUR helper or using `makepkg -i` will be of help when you update your system.

Before you update - you can query pacman for the list - pipe it to a text file for later referral

    pacman -Qm > ~/aur-pkglist.txt && xdg-open aur-pkglist.txt

## What happens on update?
---
Most of us has been using Windows and we know from Windows that system files may be locked an the system must be restarted to the file can be replaced with the updated versions.

Due to the modularity of Linux only needed portions of an application is loaded into memory and the rest is loaded on as-needed-basis - and this includes kernel, kernel modules and drivers.

When you build a package using an AUR helper the package is build using the - at build time - available system libraries. When you update your system - you are potentially replacing those dependencies with new libraries - which in turn may influence your system stability - especially when it comes to a kernel module build for a specific kernel.

When you reboot your system after an update the system will usually use the updated kernel upon reboot - and the device relying on a custom kernel module will fail because the module has not been rebuild.

Only after a successful update and a successful restart you can begin to update your custom modules.

## What happens when an update stalls?
---
The following is a simplified description - of what may happen.

When you run huge updates within the graphical environment - whether you use a terminal or a gui package manager - the system may end in a state where it needs to load a specific library at a specific memory address pointing at a specific routine in the library - but due to updates - the library is no longer existing - it has been updated to a never version - and your update stalls.

## Updating the smart way
---
Updating the smart way includes knowing which packages may pose a problem if the update is run with the GUI. I showed you a list of packages which by a lot of users experience often requires special attention.

When Pamac shows you a huge list of update - it can be hard to get a clear view and you could miss one.

You can use a small [utility script][1] simplifying the task of evaluating a huge list of updates.

When you have evaluated the list and found this update to have the potential to stall - the smart way is to update using the console. Logout of your GUI and switch to TTY.

### Ensure mirror is up-to-date
Before switching to console and executing the update command - ensure your primary mirror is up-to-date
```bash
$ pacman-mirrors
Pacman-mirrors version 4.16.4
Local mirror status for unstable branch
Mirror #1   OK  00:31   Denmark  https://www.uex.dk/public/manjaro/
```
### Updating
Logout of your GUI and switch to TTY- <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F4</kbd> and run the update. Depending on the result you may proceed to update the system

```bash
$ sudo pacman -Syu
```
If you need to rebuild your mirror list - you will need to use below variation of the update command - forcing your system package database to equal your primary mirror's databases and as pointed to by @cscs Pamac has a timer - rebuilding your mirror list on a regular base - in which case the this command should be favored.

```bash
$ sudo pacman -Syyu
```
When you reboot after a successful update - first thing to do is rebuilding your foreign packages, drivers and kernel modules. This can be done from a terminal in the GUI session - but if you want to ensure you get no problems - TTY is recommended <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F4</kbd>. 

Depending on your DE you switch back to GUI using <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F7</kbd> or <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F1</kbd>

Rebuilding all your AUR packages using a single command e.g.
```bash
$ pamac upgrade -a
```

## Conclusion
Following a few basic rules will help you keep your system healthy and hopefully you will avoid problems from a stalled update.



[1]: https://forum.manjaro.org/t/root-tip-check-if-updates-may-require-system-restart/14112