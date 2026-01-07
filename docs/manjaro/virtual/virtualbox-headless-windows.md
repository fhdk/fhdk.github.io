---
title: 'Headless Windows on VirtualBox'
taxonomy:
    category:
        - docs
date: '20-04-2020 08:40'
---

## Running a headless Windows using VirtualBox and systemd

What I had in mind was a powerhouse - to run a Windows 10 with Visual Studio and SQL server - as a virtual machine - a development project I have.

I want to power on the system and after boot - ideally - the Windows system would be available using RDP.

After some speculations I decided to look at my pile of hardware - I got a few servers collecting dust - but they are noisy as a highway at rush hour. 

But I got a tower from when my daughter was teenager. Its an older i7 on an Intel board - maybe it will do.

```
System:    Host: tower Kernel: 5.5.13-1-MANJARO x86_64 bits: 64 Console: tty 0 
           Distro: Manjaro Linux 
Memory:    RAM: total: 15.63 GiB used: 566.3 MiB (3.5%) 
           Array-1: capacity: 32 GiB note: est. slots: 4 EC: None 
           Device-1: CHAN A DIMM 0 size: 8 GiB speed: 1067 MT/s 
           Device-2: CHAN A DIMM 1 size: No Module Installed 
           Device-3: CHAN B DIMM 0 size: No Module Installed 
           Device-4: CHAN C DIMM 0 size: 8 GiB speed: 1067 MT/s 
CPU:       Topology: Quad Core model: Intel Core i7 950 bits: 64 type: MT MCP L2 cache: 8192 KiB 
           Speed: 1825 MHz min/max: 1596/3060 MHz Core speeds (MHz): 1: 2389 2: 2278 3: 2029 
           4: 1993 5: 1621 6: 1916 7: 1962 8: 1696 
```

---
I have a virtual machine with Visual Studio and SQL installed but that is on another system - what would be the easiest way to directly transfer the VM 127G? NFS of course. 

I added an extra 250G SSD for VM storage to the system and enabled/started the ssh daemon so I could continue using terminal.

Added the necessary setup - using my own [guide to NFS][1] (my memory is muscle memory - I haven't setup that much NFS). I then transferred the VM to the new system.

Installed the virtualbox packages - build the oracle extension pack (necessary for VRDE).

VirtualBox looks for machines in the user's **~/VirtualBox\ VMs** folder so I made a symlink from the VM storage location into my user home

    ln -s /data/virtualbox ~/VirtualBox\ VMs

[VirtualBox CLI][2] is - according to Oracle - the only method exposing all possible VM manipulation you think of.

VirtualBox cannot start a virtual machine if it don't know of it - so I needed to register the VM

    VBoxManage registervm /home/$USER/VirtualBox\ VMs/vstudio/vstudio.vbox

To be able to access the VM remotely - I need to enable VRDE (virtualbox-ext-oracle contains this functionality).

    VBoxManager modifyvm vstudio --vrde on

Verify

    VBoxManage showvminfo vstudio | grep VRDE

Start the VM

    VBoxManage startvm vstudio --type headless

On my workstation I already have **remmina** and **freerdp** packages - so to verify have set it up - launching remmina and connecting to my tower IP - and login to the Windows 10 system.

    VBoxManager controlvm --savestate

---
Now auto starting the VM should be a piece of cake - but everything I can think of is scripted and it requires a login - and I would very much like to avoid that.

After more thinking - well - others must have had this idea too - and there is. I found a very old post on using [systemd to load a virtual machine on boot][3].

If was incredibly simple - and honestly - I doubted it would work - because the VM is stored in my users home - but to my pleasant surprise - it works :slight_smile:

The topic mentioned the requirement of the user owning the machine and responsible for starting it should be member of **vboxusers** group.

```
[Unit]
Description=VBox Virtual Machine %i Service
Requires=systemd-modules-load.service
After=systemd-modules-load.service

[Service]
User=user
Group=vboxusers
ExecStart=/usr/bin/VBoxHeadless -s %i
ExecStop=/usr/bin/VBoxManage controlvm %i savestate

[Install]
WantedBy=multi-user.target
```
I discovered VBoxHeadless and that you can enable VRDE when using VBoxHeadless to start the VM.

Saved the file as **vboxvmservice@.service**

Enable a vm specific service

    sudo systemctl enable vboxvmservice@vstudio.service

Start the service

    sudo systemctl start vboxvmservice@vstudio.service

I also discovered that the defaults for such systemd service may timeout and the service shutdown before the VBoxManage command could finalize the saving of state - making Windows start in trouble shooting mode.

Adding a **TimeoutStopSec=60** to the **Service** definition solved the issue. If you experience Windows Repair issues because it takes longer to save the state of the VM - increase the value.

It is also possible to use **acpipowerbutton** instead of **savestate**. Be careful with that if you don't have an up-to-date installation of the guest-iso inside the VM. If not the acpipowerbutton is not reliable.

---
So now I power on the ~server~ computer - wait until it is up - then I connect to my visual studio virtual machine and I have released my workstation of that burden.

And I can shut it down without damaging my windows installation.


[1]: https://archived.forum.manjaro.org/t/howto-share-data-between-two-computers-using-nfs/102992?u=linux-aarhus
[2]: https://www.oracle.com/technical-resources/articles/it-infrastructure/admin-manage-vbox-cli.html
[3]: http://www.ericerfanian.com/automatically-starting-virtualbox-vms-on-archlinux-using-systemd/