---
title: 'Make Manjaro compatible with VPN providers'
date: '06:55 14-02-2025'
taxonomy:
    category:
        - docs
---

## VPN compatibility

Major VPN providers offer a GUI application which handles all aspects of the connection.

Every now and then the topics on troubleshooting a given VPN provider surfaces and a lot of topics boils down to DNS and for good reason.

Whether you are using an app offered by your provider or you are using configuration files it is of utmost importance you ensure correct configuration of DNS.

## Manjaro default resolver

Since the dawn of Manjaro, the network has been configured using NetworkManager and resolved provided by **openresolv** package.

## VPN provider expectations

All VPN providers maintains compatibility with Ubuntu, Debian and Fedora and as they all use systemd-resolved as DNS resolver, in fact they expect it to be so.

This makes it a - at times, hairpulling - challenge to get the VPN into an functional state where DNS work both before and after connection.

## Provider provisioned apps

There is a helper package **systemd-resolvconf** which provides a symlink to resolvctl in case you are used to **resolvconf** on the commandline or an older application expects resolvconf to be a working binary.

> However, if the [DHCP](https://wiki.archlinux.org/title/DHCP) and [VPN](https://wiki.archlinux.org/title/VPN) clients use the [resolvconf](https://en.wikipedia.org/wiki/resolvconf) program to set name servers and search domains (see [openresolv#Users](https://wiki.archlinux.org/title/Openresolv#Users) for a list of software that use *resolvconf*), the additional package [systemd-resolvconf](https://archlinux.org/packages/?name=systemd-resolvconf) is needed to provide the `/usr/bin/resolvconf` symlink.
> -- https://wiki.archlinux.org/title/Systemd-resolved#Automatically

The process is simple

1. backup your resolv.conf
2. enable systemd-resolved
3. create a symlink linking the stub-resolv.conf as resolv.conf
4. uninstall openresolv
5. install systemd-resolvconf

Copy and paste these commands and you are done

```
sudo cp /etc/resolv.conf /etc/resolv.conf.bak
sudo systemctl enable --now systemd-resolved
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
sudo pacman -Rns openresolv
sudo pacman -Syu systemd-resolvconf

```

If you want to provide custom DNS entries you can do so in **/etc/systemd/resolved.conf**

## OpenVPN

TODO

## Resources

- [https://wiki.archlinux.org/title/Systemd-resolved](https://wiki.archlinux.org/title/Systemd-resolved)
- [https://man.archlinux.org/man/resolvectl.1](https://man.archlinux.org/man/resolvectl.1)
- [https://man.archlinux.org/man/resolved.conf.5](https://man.archlinux.org/man/resolved.conf.5)
- [https://forum.manjaro.org/t/81016](https://forum.manjaro.org/t/81016)
