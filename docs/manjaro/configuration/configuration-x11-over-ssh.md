---
title: 'X11 over SSH'
date: '05:54 06-08-2023'
taxonomy:
    category:
        - docs
---

# Configure SSH to forward X

## Editing server config
Server configuration is found in **`/etc/ssh/sshd_config`**

### X11 forward for specified user
~~~
Match User manjaro
  X11Forwarding yes
~~~

### Configure display on host
Default :10
~~~
X11DisplayOffset 10
~~~

### If you want to use all display change
~~~
X11UseLocalhost no
~~~

## Client
Client configuration is stored in **`~/.ssh/config`**
### Connect
~~~
ssh -X manjaro@host.name
~~~

### Alternative connection method
~~~
ssh -o ForwardX11=yes manjaro@host.name
~~~

### Make X11 forwarding a permanent option.
Edit the local config and set
~~~
X11Forwarding yes
~~~

## Running an X11 app over ssh
e.g. xclock (fork to background)
~~~
ssh -X -f user@host xclock
~~~

For more info read the man page
~~~
man ssh
~~~


## Troubleshooting
check xauth
~~~
 $ which xauth
/usr/bin/xauth
~~~
It is usually present on Manjaro - if not sync the package
~~~
sudo pacman -S xorg-xauth
~~~