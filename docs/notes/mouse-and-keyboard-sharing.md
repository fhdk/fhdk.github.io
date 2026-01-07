---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Mouse and keyboard sharing'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Share mouse and keyboard
Many users of Linux have several devices in use - different purposes.

Havíng a laptop and workstation in use at the same time requires you to move your attention from one keyboard to another - from one screen to another.

What if it was possible to use your workstations mouse and keyboard to operate your laptop?

## Synergy
Such solution exist - [Synergy](https://symless.com/synergy). Unfortunately this is partly a commercial solution but based on Open Source code so the Github user [debauchee](https://github.com/debauchee/barrier#barrier) has forked the code and created a rebranded fully Open Source solution which can be installed from AUR.

## Barrier
> ### What is it?
>
>Barrier is KVM software forked from Symless's synergy 1.9 codebase. Synergy was >a commercialized reimplementation of the original CosmoSynergy written by Chris Schoeneman.
>
>### What's different?
>
>Whereas synergy has moved beyond its goals from the 1.x era, Barrier aims to >maintain that simplicity. Barrier will let you use your keyboard and mouse from >machine A to control machine B (or more). It's that simple.

## Installation
Install the package from AUR on both systems with Pamac using either the GUI or the CLI

    pamac build barrier

## Naming terms
* The ***server*** is the system offering mouse and keyboard service
* The ***client*** is the computer to be controlled by the server

## Configure the server
Launch the barrier app from your environments menu system.

 1. Select the **Server** checkbox
 2. Click the button **Configure Server...**
 3. In the center of the grid you see your server computer by name.
 4. Drag the monitor from the upper left corner and place in a logical position in relation to your ***server***. E.g. if your ***client*** is located on left side of your ***server*** placed it the left in grid.
 5. Double click the newly placed ***client***, name it and click **OK**.
 6. Make a note of the SSL Fingerprint show in the **Server** box. You will need it when connecting the ***client** for the first time.
 7. Click **Apply** and **Start** to start the ***server***

Note: you can setup several ***clients*** if you have more than one you want to control. Only requirement is a monitor attached to ***client***.

Save the server configuration to a file using the **Barrier** menu or press <kbd>Ctrl</kbd><kbd>Alt</kbd><kbd>s</kbd>, name the file **.barrier.conf**.

### Run barrier server at login
If you want to run the barrier server at login you need to create a script and add this to your environments autostart.

Use your preferred method of editing text files to create a script named **barrier.sh** and make it executable.

    touch ~/barrier.sh
    chmod +x ~/barrier.sh
    nano ~/barrier.sh

Add the following content to the file

    SERVERNAME=$(hostname)
    CONFIG=/home/$USER/.barrier.conf
    barriers --no-tray \
             --debug INFO \
             --name $SERVERNAME \
             --enable-crypto \
             -c $CONFIG \
             --address :24800

If your environment uses a autostart file e.g. Openbox add the **`barrier.sh`** script to your autostart file
```
nano ~/.config/openbox/autostart
```
```
sleep 1; ~/barrier.sh
```
## Configure the client
The first connection from your ***client*** has to be done manually as the ***server*** 's SSL fingerprint will have to be manually acknowledged before use.

### Run barrier client
Like we did on the ***server***, we will create a script to launch the client at login. Replace **serverip** with the IP of your ***server***. Do not remove the brackets, they are part of the argument.
```
touch ~/barrier.sh
chmod +x ~/barrier.sh
nano ~/barrier.sh
```
```
CLIENTNAME=$(hostname)
barrierc --no-tray \
         --debug INFO \
         --name $CLIENTNAME \
         --enable-crypto \
         [serverip]:24800
```
As with the server add the script to your environment's autostart. If the environment is Openbox add the **`barrier.sh`** script the clients autostart file
```
nano ~/.config/openbox/autostart
```
```
sleep 1; ~/barrier.sh
```
Setup the ***client***'s display manager to auto login. If you are using lightdm then install the package `lightdm-settings` and configure auto login and restart the client.

Log off the server and re-login.
