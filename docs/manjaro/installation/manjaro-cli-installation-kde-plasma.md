---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'CLI to LXDE'
taxonomy:
    category:
        - docs
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

For this excercise I have chosen [LXDE](https://lxde.org/) because it is lightweight and offers a familiar Windows like interface. I have chosen Midori because of the small footprint but it could as well be Firefox or Chromium.

When you get the hang of it you can modify to your liking but for now install the below packages using **pacman**.  You may need to research if you need WiFi driver packages not mentioned below.

[details="Xorg packages"]
* xorg-server
* xorg-server-common
* xorg-xinit
* xorg-drivers
[/details]

[details="Wireless packages"]
* ipw2100-fw
* ipw2200-fw
[/details]

[details="Lxde vanilla"]
* accountsservice
* gnome-keyring
* gnome-icon-theme
* gnome-themes-standard
* lxappearance
* lxde-common
* lxde-icon-theme
* lxhotkey
* lxinput
* lxlauncher
* lxmed
* lxmenu-data
* lxpanel
* lxsession
* lxtask
* lxterminal
* obconf
* pcmanfm
* perl-file-mimeinfo
* xdg-user-dirs
* xdg-user-dirs-gtk
* xdg-utils
[/details]

[details="Network"]
* midori
* network-manager-applet
[/details]

[details="Manjaro theming"]
* lxde-wallpapers
* manjaro-lxde-config
* manjaro-lxde-desktop-settings
* manjaro-lxde-logout-banner
* matcha-gtk-theme
* manjaro-openbox-matcha
* papirus-icon-theme
* papirus-maia-icon-theme
* ttf-dejavu
* ttf-roboto
* xcursor-breeze
[/details]

```bash
# pacman -Syu xorg-server xorg-server-common xorg-xinit xorg-drivers ipw2100-fw ipw2200-fw iw iwd accountsservice gnome-keyring gnome-icon-theme gnome-themes-standard leafpad lxappearance lxde-common lxde-icon-theme lxhotkey lxinput lxlauncher lxmed lxmenu-data lxpanel lxsession lxtask lxterminal manjaro-icons obconf pcmanfm perl-file-mimeinfo xdg-user-dirs xdg-user-dirs-gtk xdg-utils midori network-manager-applet lxde-wallpapers manjaro-lxde-config manjaro-lxde-desktop-settings manjaro-lxde-logout-banner matcha-gtk-theme manjaro-openbox-matcha papirus-icon-theme papirus-maia-icon-theme ttf-dejavu ttf-roboto xcursor-breeze
```

After you have installed the Xorg and Lxde packages - create a new adminstrative user. The explanation for waiting with user creation is the default settings which, on user creation, are copied from `/etc/skel` to the new users home.

```
# useradd -mUG lp,network,power,sys,wheel newuser
```
Set a password for the new user
```
# passwd newuser
```
Logout
```
# exit
```
## Start X
---
Login with the new username and launch X
```bash
$ startx
```

## Variations
---
* https://archived.forum.manjaro.org/t/howto-install-manjaro-using-cli-only/108203?u=linux-aarhus
* https://archived.forum.manjaro.org/t/howto-run-manjaro-on-a-stick/109520?u=linux-aarhus
* https://archived.forum.manjaro.org/t/howto-install-encrypted-manjaro-using-cli/110553?u=linux-aarhus
