---
title: 'CLI to vanilla Gnome'
taxonomy:
    category:
        - docs
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
metadata:
    author: linux-aarhus
---

# Extend your Manjaro CLI
---

This guide covers the extension of a basic CLI to a graphical environment using a minimal set of packages. If you want to read up on the CLI it is available on the link below.

Boot your device and login into the system using the root account.

## Install packages
---
We will now extend our basic CLI to a graphical environment using the least amount of packages and we will add in a web browser application.

When you get the hang of it you can modify to your liking but for now install the below packages using **pacman**.  You may need to research if you need WiFi driver packages not mentioned below.

```bash
# pacman -Syu xorg-server xorg-server-common xorg-xinit xorg-drivers accountsservice gnome-keyring gnome-icon-theme gnome-themes-standard gnome-session gnome-shell gnome-desktop gnome-terminal gdm
```

After you have installed the packages - create a new administrative user.
```
# useradd -mUG lp,network,power,sys,wheel newuser
```
Set a password for the new user
```
# passwd newuser
```
Enable gdm display manager
```
# systemctl enable gdm
```
Reboot
```
# reboot
```

## Variations
---
* https://archived.forum.manjaro.org/t/howto-install-manjaro-using-cli-only/108203?u=linux-aarhus
* https://archived.forum.manjaro.org/t/howto-run-manjaro-on-a-stick/109520?u=linux-aarhus
* https://archived.forum.manjaro.org/t/howto-install-encrypted-manjaro-using-cli/110553?u=linux-aarhus
