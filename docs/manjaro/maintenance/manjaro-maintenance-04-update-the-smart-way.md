---
title: 'Update Manjaro the smart way'
date: '10:03 02-11-2022'
taxonomy:
    category:
        - docs
---

linux-aarhus | 2022-10-07 09:16:16 UTC | Source: [Update Manjaro the smart way](https://forum.manjaro.org/t/root-tip-how-to-update-manjaro-the-smart-way/30979)

## Manjaro updates
---
Either when you use your system or shortly after login - a message appears - telling you there is updates available. When you click the message Pamac opens with the updates pane listing 100's of packages ready to download/install on your system. 

Almost per instinct you click the update now - provide your password and the flow begin. 

Suddenly your screen goes blank - you wait - nothing happens. 

You press <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F4</kbd> to access TTY - nothing happens - you wait - nothing happens. 

Without giving much thought to it - you hold the power button until the system powers off and press it again to power on your system - and now you are having a black screen maybe a message .... unless your system is not booting at all !

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
```bash
$ pacman -Qm > ~/aur-pkglist.txt && xdg-open aur-pkglist.txt
```

### package download
The way pacman works - using the alpm library - is to download the packages and verify by signatures - and when a package validation fail - the transaction will be not committed.

This ensures no updates are written to the system if the internet connects is severed as the update will be cancelled unless all packages required is available in the local package cache.

### pacman hooks
A part of knowing your system is to understand how the alpm library applies the updates.

Many packages - especially system packages - makes use of hooks to process changes during an update.

If a package update is not fully completed - eg. your system loses power during update - the post-update hooks are not executed - and bad things can happen - what actually happens depends on the package and the hooks implemented.

## What happens on update?
---
Most of us has been using Windows and we know from Windows that system files may be locked so the system must be restarted so the file can be replaced with the updated versions.

Due to the modularity of Linux only needed parts of an application is loaded into memory and the rest is loaded on as-needed-basis - and this includes kernel, kernel modules and drivers.

## Custom packages
When you build a custom package using an AUR helper, the package is built using the - at build time - available system libraries. 

When you update your system - you may be replacing those dependencies with new libraries - and such replacement may cause instability or outright failure of your custom package - especially when your custom packages depends on a specific kernel release.

When you reboot your system after an update the system will use new modules and any application or device relying on a custom kernel module will fail because the module has not been rebuilt.

Only after a successful update and a successful restart you can begin to update your custom modules.

### custom themes
Themes is not only a bunch of color definitions and icons.

Gnome (GTK+) and Plasma (QT+) themes is high level code integrating with the tool kit using tool kit specific definitions, code abstraction even javascript.

This makes applications prone to errors when themes are not maintained to match the next release of the tool kit. 

Such maintenance may appear tedious but nonetheless - a non maintained theme may break the system - most notably the display manager most notably __gdm__ for Gnome and __sddm__ for Plasma but also toolbars, extensions and plasmoids may exhibit erratic behavior.

## What happens when an update stalls?
---
The following is a simplified description - of what may happen.

When you run huge updates within the graphical environment - whether you use a terminal or a gui package manager - the system may end in a state where it needs to load a specific library at a specific memory address pointing at a specific routine in the library - but due to updates - the library is no longer existing - it has been updated to a never version - and your update stalls.

## Updating the smart way
---
Before major updates, it is wise to - temporarily 

- Gnome 
  - disable extensions 
  - revert to default shell theme
  - revert to default theme
- Plasma
  - revert to the default global theme (Breeze/Breath) (include desktop layout)
  - revert display manager (sddm) to the default theme (Breeze/Breath).

Updating the smart way includes knowing which packages may pose a problem if the update is run with the GUI. I showed you a list of packages which by a lot of users experience often requires special attention.

When Pamac shows you a huge list of update - it can be hard to get a clear view and you could miss one.

You can use a small [utility script][1] simplifying the task of evaluating a huge list of updates.

When you have evaluated the list and found this update to have the potential to stall - the smart way is to update using the console. Logout of your GUI and switch to TTY.

### Don't ever interrupt the process
Don't ever interrupt the process - intentionally or unintentionally.

Ensure you are not doing your update when your area are known to power spike,  or during stormy weather which can cause interruptions to the power grid - if your system loses power you will probably have a problem getting it back up and runnning.

### Ensure mirror is up-to-date
Before syncronizing your system optionally ensure your primary mirror is up-to-date

```bash
$ pacman-mirrors
Pacman-mirrors version 4.16.4
Local mirror status for unstable branch
Mirror #1   OK  00:31   Denmark  https://www.uex.dk/public/manjaro/
```

### Ensure enough diskspace
If you don't regularly clean up you system - now is a good time

Remove old entries from the journal (reduce the size to 50M)
```bash
$ sudo journalctl --vacuum-size=50M
```
Cleanup your pacman cache (this will remove uninstalled packages and reduce pkg versions to the installed version)
```bash
$ sudo paccache -ruk0
```
### Updating
Logout of your GUI and switch to TTY- <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F4</kbd> and run the update. Depending on the result you may proceed to update the system

If you are using __dkms__ based drivers - ensure they are rebuild without errors.

```bash
$ sudo pacman -Syu
```
If you need to rebuild your mirror list - you will need to use below variation of the update command - forcing your system package database to equal your primary mirror's databases and as pointed to by @cscs Pamac has a timer - rebuilding your mirror list on a regular base - in which case this command should be favored.

```bash
$ sudo pacman -Syyu
```

When you reboot after a successful update - first thing to do is rebuilding your foreign packages and drivers. This can be done from a terminal in the GUI session - but if you want to ensure you get no problems - TTY is recommended <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F4</kbd>. 

Depending on your environment, you switch back to GUI using <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F7</kbd> or if your displaymanager is SDDM <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>F1</kbd>

Rebuilding all your AUR packages using a single command e.g.
```bash
$ pamac upgrade -a
```

## After succesful sync
Reapply your custom theming and give yourself a :clap:  - you deserve it.

## Conclusion
Following a few basic rules will help you keep your system healthy and hopefully you will avoid problems from a stalled update.



[1]: https://forum.manjaro.org/t/root-tip-check-if-updates-may-require-system-restart/14112

-------------------------
