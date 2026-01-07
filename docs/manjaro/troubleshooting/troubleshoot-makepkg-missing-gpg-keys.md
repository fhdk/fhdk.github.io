---
title: 'Retrieve missing GPG using makepkg'
taxonomy:
    category:
        - docs
---

Original posted by [@FadeMind][1] on Manjaro forum

## Issue

> Some files are signed with developers GPG sign. Without validation of source tarbals, build package fails. For automatically downloading and adding new keys to LOCAL GPG database follow once this guide.

## Install GnuPG

```
sudo pacman -S gnupg pinentry --needed --noconfirm
```

## Enable following services sockets as normal user:

```
systemctl --user enable gpg-agent.socket
systemctl --user start gpg-agent.socket
systemctl --user enable dirmngr.socket
systemctl --user start dirmngr.socket
```

## Files and content

### $HOME/.gnupg/dirmngr.conf

```text
keyserver hkps://pgp.mit.edu
keyserver hkps://hkps.pool.sks-keyservers.net
keyserver hkp://keyserver.ubuntu.com:80
```

### $HOME/.gnupg/gpg.conf

```text
keyserver hkps://pgp.mit.edu
keyserver hkps://hkps.pool.sks-keyservers.net
keyserver hkp://keyserver.ubuntu.com:80
keyserver-options auto-key-retrieve
require-cross-certification
keyring /etc/pacman.d/gnupg/pubring.gpg
use-agent
```

### $HOME/.gnupg/gpg-agent.conf
> NOTICE:  If You are in  **NON-KDE**  use  `pinentry-qt`  or  `pinentry-gtk-2`  .

```
default-cache-ttl 300
max-cache-ttl 999999
pinentry-program /usr/bin/pinentry-qt

### uncomment GTK2 or kwallet variant instead of QT if you needed.
# pinentry-program /usr/bin/pinentry-gtk-2
# pinentry-program /usr/bin/pinentry-kwallet
```

pinentry-kwallet is part of  **kwalletcli**  [from AUR ][2] You need build and install (via yay/pacaur tool) if you want store passwords in kwallet.

## Reload GPG Agent

```
gpg-connect-agent reloadagent /bye
```


[1]: https://forum.manjaro.org/t/howto-automatically-retrieve-missing-gpg-keys-during-making-packages-20201019/32844
[2]: https://aur.archlinux.org/packages/kwalletcli