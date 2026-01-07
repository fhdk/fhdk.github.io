---
title: 'TigerVNC over SSH'
taxonomy:
    category:
        - docs
---

## VNC 
VNC is a protocol where you use your keyboard/mouse/screen an mouse to monitor or control a remote system.

This document is a real-world implementation of the document(s) found at [Archlinux Wiki - TigerVNC][1].

__NOTE__: The setup is not implementing any VNC encryption so SSH will be used to establish the connection.

The benefit of using SSH is that you can easily adapt this to target any remote server.

## Target system
On the system to be controlled install package `tigervnc`

### Config
Replace the phrase `$USERNAME` with the username of the actual user you want to configure.

Create the file `~/$USERNAME/.vnc/config` with content (_replace session according to your installation_)
```text
session=openbox
geometry=1400x900
localhost
dpi=96
```
Create a password for the user using `vncpasswd`

Edit `/etc/tigervnc/vncserver.users` and append
```
:2=$USERNAME
```
Start a vncserver at the selected display
```
systemctl enable --now vncserver@:2
```

## Controlling system
Install the package `tigervnc`

### Connect to target system
SSH provides a secure channel and using key based authentication is the recommended method.

Open a ssh connection using port mapping
```
ssh $USERNAME@client -L 9902:localhost:5902
```
Then launch the VNC viewer and input the following connection details then click connect

> localhost:9902

Input the password created earlier (ignore the warning as we are using an encrypted connection) and you will see the remote system.



[1]: https://wiki.archlinux.org/title/TigerVNC
