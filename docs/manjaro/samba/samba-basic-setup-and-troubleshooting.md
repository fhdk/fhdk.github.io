---
title: 'Basic Samba Setup and Troubleshooting'
taxonomy:
    category:
        - docs
---

**Difficulty: ★★☆☆☆**

What is SAMBA
===

> Samba is the standard Windows interoperability suite of programs for Linux and Unix.

Content
---

* [Occasional sharing][100]
* [Samba Client][110]
* [GVFS][120]
* [System Automount][130]
* [User Mount][140]
* [User Automount][150]
* [User Service][151]
* [Manjaro Samba Server][160]
* [Troubleshooting][170]
* [References][180]

What can you learn from this?
---
This document from my [notepad][0] is intended as a help to setup __simple__ client connection to __home NAS__ and simple file sharing from your Manjaro desktop to other computers in your __home network__. If you are looking for domain specific setup and integration to Microsoft AD you must refer to the official samba documentation<sup>[[1]]</sup>. 

## Occasional sharing
---
### Python http.server
If you only occasionally need to serve files - you can do so using the default Python installation.

Copy the files to you want to share to __~/Public__ then open a terminal in the folder and run two commands. 

The first displays your IP address <sup>[utility script [2]]</sup>

    ~/Public $ check-network
    192.168.30.20

The second run the http service exposing the content of your `~/Public` folder

    ~/Public $ python -m http.server -d ~/Public  8080 
    Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...

Then share the IP and port. The person(s) can fetch the file(s) using their browser and navigating to - the slash at the end is important (otherwise the browser runs a search using the configured searchengine)

    http://$IPADDRESS:8080/

When you are finished sharing, close the terminal or press <kbd>Ctrl</kbd><kbd>c</kbd> to stop sharing.

### Simple http servers
Packages such as __darkhttpd__ and __caddy__ provides webservers to service static files and can be installed from the repo.

The __caddy__ package also provides a SSL/TLS enabled server.


## Samba Client
---
For simple client connection to shares provided by a NAS or maybe your router only a few packages are needed. The package `smbclient` provides the tools necessary for accessing samba fileshares.

There is several packages which builds upon the client and makes it easier to connect and automount a share from a file manager. Packages such as `gvfs-smb` and `smb4k` extends the file manager with seamless mount of Samba shares.

Ensure the packages are up-to-date by running below command in a terminal

    sudo pacman -Syu smbclient gvfs gvfs-smb --needed

### smbclient
Before attempting a connection either reboot your system or manually load the kernel module

    sudo modprobe cifs

Also a configuration file `/etc/samba/smb.conf` must be present. The client doesn't require any content - just the presence and can thus be created by the `touch` utility.

    sudo mkdir /etc/samba
    sudo touch /etc/samba/smb.conf

## GVFS
---
Open your file manager and enter the server name or IP address in location bar. 

If the location bar is not visible it is often activated using the hot key-combo <kbd>Ctrl</kbd><kbd>L</kbd>. Input the server name and share using the protocol format

```text
smb://server-name/share
```
Some file managers (e.g. dolphin) can better deduct if credentials are required if the username is supplied like this

```text
smb://username@server-name/share
```

When you are challenged for credentials input those to access the content. Do not save your credentials just yet. Later on you will return here  to store the credentials in  your keyring.

This method is using gvfs which creates the mountpoint in the __/run__ tree and therefore the mount will not persist across reboot. 

You can inspect the file structure by opening the folder matching the uid for your user. If you have a mounted share you can run `ls` on the gvfs folder - it will yield an output similar to below example

```
$ ls /run/user/$UID/gvfs
'smb-share:server=nas.net.nix.dk,share=data'
```

## System Automount
---
### systemd units
When mounting one or more network shares on boot one need to take into account when the network is up and connected - otherwise the system will hang for 90s for each share.

This can be done using fstab or using systemd units <sup>[[ systemd unit ][3]] [[ sample units ][4]]</sup>.

## User Mount
As demonstrated, a file manager can utilize gvfs to create a user mounted samba share, and this knowledge can used in creating some automation.

As shown the gio mountpoint takes the following form where `$UID`, `$HOST` and `$SHARE` represents the variable factors
```
/run/user/$UID/gvfs/'smb-share:server=$HOST,share=$SHARE'
```

We will use the __[gio smb mount][8]__ script to mount the share. Use the template provided in the topic to create your own script. When you have the script in place and working come back here.

The script asks for username, workgroup and password when you run it and that is fine for the occasional mount but you will likely want to automount the share when you log onto your system.

## User Automount
The script can be executed at login in a number of ways - but remember - the script's success depends on the credentials being available in your keyring.

* Adding the script to your environment's autostart configuration
* Manually creating a desktop launcher in `~/.config/autorun`

This topic will propose a somewhat different method by means of a __systemd user service__ which will be used to execute our script upon login.

:spiral_notepad: To be able to automount the share you can

__Either__
* Stored the credentials in your keyring when challenged by the file manager.

__Or__
* Store the credentials in the mount script mentioned above

## The user service
Create the folder `~/.config/systemd/user` and create a service file named e.g. `gio-smb-share-name.service` - use the same name as the script you are calling - thus making the dependency obvious and simplify future maintenance.

    mkdir -p ~/.config/systemd/user
    touch ~/.config/systemd/user/gio-smb-share-name.service

Open the file in your favorite editor and paste below content.

```
[Unit]
Description=GIO mount smb share-name

[Service]
Type=oneshot
ExecStart=/home/%u/.local/bin/gio-smb-share-name.sh
ExecStop=/home/%u/.local/bin/gio-smb-share-name.sh -u
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```
Watch your SMBLinks folder and start the service

    systemctl --user start gio-smb-share-name.service

Note the symlink is created. Stop the service to watch the symlink disappear

    systemctl --user stop gio-smb-share-name.service

When everything works - start and enable the user service

    systemctl --user enable --now gio-smb-share-name.service


## Manjaro Samba Server
---
:warning: 
1. Sharing data will __open ports to the network__. 
2. Only run a Samba file service on a __trusted secure network or zone__.
3. __Ensure firewalld__ is set up to only allow share traffic when connected to a trusted and secure network or zone.
4. __Never__ run a samba file server on a public network.
5. __Avoid NT1 aka SMB1__ - it is unsafe and exploited by numerous ransomware projets.

### Secure your service
if you are using another firewall service (e.g. ufw) either setup that service to allow samba and skip this step __or__ stop and disable the service before continuing.

Install the package __firewalld__ then enable and start the service.

```
sudo pacman -Syu firewalld
sudo systemctl enable --now firewalld
```
firewalld has some predefined zones which we will use for this purpose to define trusted and untrusted network. If a network is not defined in any zones it is treated as __public__ which means visible on the network and no open ports

* home
* public

When you are connected to what you deem a trusted and secure network run the following command

```
nmcli device show | grep IP4.ROUTE
IP4.ROUTE[1]:             dst = 192.168.1.0/24, nh = 0.0.0.0, mt = 100
IP4.ROUTE[2]:             dst = 0.0.0.0/0, nh = 192.168.1.1, mt = 100
```
In this example note the first destination network; __192.168.1.0/24__. This it a common network used by many home routers. Your result may be different - adjust accordingly.

Before you begin - list the default services permanently enabled for the zones
```
sudo firewall-cmd --permanent --zone="home" --list-services
sudo firewall-cmd --permanent --zone="public" --list-services
```

Next step is to to tell the firewall this network is trusted - replace the variable with your network (note it is possible to have more than one trusted network - eg. home and work)
```
sudo firewall-cmd --permanent --zone="home" --add-source="$NETWORK"
```
Example using above 
```
sudo firewall-cmd --permanent --zone="home" --add-source="192.168.1.0/24"
```

Next we define the policy for the samba service
```
sudo firewall-cmd --permanent --zone="home" --add-service="samba"
```

With the security in place proceed to installing and configuring Samba

### Install samba package
Install the samba package and ensure your system is fully updated in the process.

```text
sudo pacman -Syu samba
```

### Basic Server configuration

Create the configuration file **/etc/samba/smb.conf** - the folder may need to be created beforehand.
```text
sudo mkdir -p /etc/samba
sudo touch /etc/samba/smb.conf
```
Edit the file - using superuser privilige - insert below content and save the file (need superuser). If you are connecting an existing network of servers change the WORKGROUP to match the existing network. 

```text
[global]
   workgroup = MANJARO
   server string = Manjaro Samba Server
   server role = standalone server
   log file = /var/log/samba/log.%m
   max log size = 50
   guest account = nobody
   map to guest = Bad Password
   
   min protocol = SMB2
   max protocol = SMB3

[public]
   path = /srv/samba/share-name
   public = yes
   writable = yes
   printable = no

```
### Test your config
```text
sudo testparm /etc/samba/smb.conf
```
### Create share and set permissions
To share e.g. removable devices you need to mount the devices. 

This is best done using using systemd mount and if removable create both mount and automount unit.

* https://forum.manjaro.org/t/root-tip-use-systemd-to-mount-any-device/1185


Create the shared folder
```text
sudo mkdir -p /srv/samba/share-name
```
Set permissions to any and all
```text
sudo chmod -R ugo+rw /srv/samba/share-name
```

### Start the services
```text
sudo systemctl enable --now smb nmb
```

## Troubleshooting
---
Did you remember to add users to the service? 

Samba users can be added independently from system users by using

    smbpasswd -a $USERNAME

* Samba documentation on https://www.samba.org/samba/docs/current/man-html/smbpasswd.8.html

### Access Control Lists aka [ACL][11]
Missing r/w access due to permissions. In essence, it was similar to [this issue][12]. Simple solution is to remove your share and create it again. And no need to change `tdbsam` to `smbpasswd`. -- Credit @openminded 

### AppArmor when serving samba shares
Some eitions of Manjaro comes with AppArmor enabled (snap support) which may lead to issues when samba is updated. In such case you can use the following quick fix
```
sudo aa-complain /etc/apparmor.d/usr.sbin.smbd
``` 
Now proceed to reading [documentation on AppArmor][13] especially the section on [editing AppArmor profiles][14] for a better and more long-term solution. -- Credit @openminded 

On [date=2022-09-03 timezone="Europe/Copenhagen"] @0xbeaf contributed this [comment][15] on another topic and has been added here to complement this section.
 
> * Changes to how `samba` does [rpc seems](https://gitlab.com/apparmor/apparmor/-/merge_requests/871) to be the main culprit. Since it is using multiple new executable files to delegate its jobs, new profiles and overrides for `apparmor` needs to be added.
> * I had to add override rules for `usr.sbin.smbd`, `samba-dcerpcd`, `samba-rpcd` and `samba-rpcd-classic`.
> * To get rid of all **DENIED** error messages in `apparmor` log, I had to add one more entity to profile rules. E.g. to whitelist the path `/tank/anbaar`, I added the following to all above-mentioned profiles in `/etc/apparmor.d/local`:
> 
> ```ini
> /tank/anbaar lrwk,
> /tank/anbaar/ lrwk,
> /tank/anbaar/** lrwk,
> ```
> * Additionally, `/usr/share/samba` and `/var/cache/samba` had to be whitelisted for `samba-rpcd` so `avahi` service could properly work with a macOS machine so that it could automatically find and mount the TimeMachine share-points!!!
> 
> Helpful links:
> 
> 1. https://ubuntu.com/server/docs/security-apparmor
> 2. https://gitlab.com/apparmor/apparmor/-/wikis/AppArmor_Core_Policy_Reference
> 3. https://bugs.archlinux.org/task/74614
> 4. https://gitlab.com/apparmor/apparmor/-/merge_requests/871

### Client
Samba client requires the file `/etc/samba/smb.conf` even if it is empty.
```
sudo mkdir -p /etc/samba
sudo touch /etc/samba/smb.conf
```

### Browse available shares
__It is not possible to browse shares by using the service name__

SMB1 has been removed for security reasons. This also removed the possibliity to browse a server for avaliable shares.

The removal enhances the resistance against ransomware and other malware propagating over the network.

```
$ smbclient -L {host|ip} -U%

        Sharename       Type      Comment
        ---------       ----      -------
        public          Disk      
        IPC$            IPC       IPC Service (Manjaro Samba Server)
SMB1 disabled -- no workgroup available
```

Netbios requests has been disabled due to the vulnerabilities thus highly improving restance against ransomware abusing the service to spread itself.

Always connect to the share directly
```
smbclient //{host|ip}/public
```

Using a file manager you may need to add username to the request e.g. __smb://username@service/sharename__ and provide the password. Only then you will be able to see what other shares the service provides.


### Initial share lookup
If you find browsing the network from your filemanager is not possible issues it may be helpful to add the following content - replace the WORKGROUP if necessary

    [global]
        workgroup = WORKGROUP

You may get messages like

> Failed to retrieve share list from server: No such file or directory
> The specified location is not mounted


### Samba protocol version
Upstream Samba disabled SMB1 due to a vulnerability exploited by a widespread ransomware. Yet many ISP provided routers has not been upgraded and you may have difficulties connecting the router's samba service using a default samba client.

To gain access to such share you need to add a samba configuration which sole purpose is to enable the deprecated samba version. Add the following line in the [global] section of the configuration file

    [global]
        client min protocol = NT1
    
### Arch Wiki
* [WINS host names][9]
* [Manual Mountiing][10] 


> The options `uid` and `gid` corresponds to the local (e.g. client) [user](https://wiki.archlinux.org/title/User)/[user group](https://wiki.archlinux.org/title/User_group) to have read/write access on the given path.
> 
> **Note:**
> 
> * If the `uid` and `gid` being used does not match the user of the server, the `forceuid` and `forcegid` options may be helpful. However note permissions assigned to a file when `forceuid` or `forcegid` are in effect may not reflect the the real (server) permissions. See the *File And Directory Ownership And Permissions* section in [mount.cifs(8) § FILE AND DIRECTORY OWNERSHIP AND PERMISSIONS](https://man.archlinux.org/man/mount.cifs.8#FILE_AND_DIRECTORY_OWNERSHIP_AND_PERMISSIONS) for more information.
> * To mount a Windows share without authentication, use `"username=*"`.
> 
> **Warning:** Using `uid` and/or `gid` as mount options may cause I/O errors, it is recommended to set/check correct [File permissions and attributes](https://wiki.archlinux.org/title/File_permissions_and_attributes) instead.
> -- Arch Wiki

## More reading
---
* My notepad <sup>[[0]]</sup>
* Samba website <sup>[[1]]</sup>
* LAN IP utility script <sup>[[2]]</sup>
* Using systemd mount units <sup>[[3]]</sup>
* Sample mount units <sup>[[4]]</sup>
* Samba on Arch Wiki <sup>[[5]]</sup>
* Sample Samba configuration with comments <sup>[[6]]</sup>
* Microsoft's documentation <sup>[[7]]</sup>
* GIO mount SMB utility script <sup>[[8]]</sup>
* AppArmor Arch Linux Wiki <sup>[[13]]</sup>

<!-- external links -->
[0]: https://root.nix.dk/en/network-notes/basic-samba-setup-and-troubleshooting
[1]: https://www.samba.org/samba/docs/
[2]: https://forum.manjaro.org/t/root-tip-utility-script-get-lan-ip-address/100603
[3]:https://forum.manjaro.org/t/root-tip-use-systemd-to-mount-any-device/1185
[4]: https://forum.manjaro.org/t/root-tip-systemd-mount-unit-samples/1191
[5]: https://wiki.archlinux.org/index.php/Samba
[6]: https://git.samba.org/samba.git/?p=samba.git;a=blob_plain;f=examples/smb.conf.default;hb=HEAD
[7]: https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-smb
[8]: https://forum.manjaro.org/t/root-tip-utility-script-gio-mount-samba-share/100723?u=linux-aarhus
[9]: https://wiki.archlinux.org/title/Samba#NetBIOS/WINS_host_names
[10]: https://wiki.archlinux.org/title/Samba#Manual_mounting
[11]: https://wiki.archlinux.org/title/Access_Control_Lists
[12]: https://www.spinics.net/lists/samba/msg173272.html
[13]: https://wiki.archlinux.org/title/AppArmor
[14]: https://wiki.debian.org/AppArmor/HowToUse
[15]: https://forum.manjaro.org/t/cant-connect-to-samba-after-update-service-is-running-2022-08-13/119396/69