---
title: 'Manjaro cloud instance using cloud-init'
taxonomy:
    category:
        - docs
---

## Minimal CLI system using VirtualBox
Use any Manjaro ISO but at the boot menu edit the free drivers item by pressing <kbd>e</kbd> and append **3** to the linux line.

When booted to the terminal - login using **root**:**manjaro**

**1. Keyboard**
If tyou need a differenct  keyboard than **us** then use **loadkeys** followed by a country code e.g. for Denmark
```bash
# loadkeys dk
```

**2. System time**
Ensure system time is correct - necessary for SSL certificates
```bash
# systemctl enable --now systemd-timesyncd
```

**3. Mirror and branch**
We use pacman-mirrors to set a mirror and the desired branch.
```bash
# pacman-mirrors --api --set-branch unstable --url https://manjaro.moson.eu
```
You can replace the branch with stable or testing and you can remove the **--url** argument and use e.g. **--continent** for closer mirrors.

**4. Database and keyrings**
Ensure pacman is up-to-date, download the databases and install keyrings
```bash
# pacman -Syy pacman archlinux-keyring manjaro-keyring
```

**5. Trust database**
Create trust database, populate and refresh keys
```bash
# pacman-key --init
# pacman-key --populate archlinux manjaro
# pacman-key --refresh-keys
```
If for some reason the above fails on the last command you need to specify a keyserver - and to mention a few

* keyserver.pgp.com
* keyserver.ubuntu.com
* pgp.mit.edu
* 
 
## 3. Partitioning disk
```
root@ubuntu:~# lsblk 
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0     7:0    0   31M  1 loop /snap/snapd/9607
loop1     7:1    0 55.3M  1 loop /snap/core18/1885
loop2     7:2    0 70.6M  1 loop /snap/lxd/16922
vda     252:0    0   25G  0 disk 
├─vda1  252:1    0 24.9G  0 part /
├─vda14 252:14   0    4M  0 part 
└─vda15 252:15   0  106M  0 part /boot/efi
```


```bash
# cfdisk --zero /dev/sda
```

### Using MBR
* Select dos partition type
* Create one partition primary
* Format
   ```bash
   # mkfs.ext4 -L MJROROOT  /dev/sda1
   ```
* Mount
   ```bash
   # mount /dev/sda1 /mnt
   ```

### Using GPT
* Select gpt
* Create a small (8MB) unformatted partition - partition type BIOS boot (0xEF02)
* Create the root partition
* Format
   ```bash
   # mkfs.ext4 -L MJROROOT /dev/sda2
   ```
* Mount
   ```
   # mount /dev/sda2 /mnt
   ```

## 6. Base installation
---
Use **basestrap** command to install a base set of packages into the newly mounted root

```bash
# basestrap /mnt base linux54 dhcpcd grub mkinitcpio vi nano sudo links openssh firewalld
```
## 7. Base configuration
---
```bash
# manjaro-chroot /mnt /bin/bash
```
### Console keyboard
Set console keyboard in **/etc/vconsole.conf** 
```bash
KEYMAP=us
```

### Locale
To generate the messages edit **/etc/locale.gen** and remove the comment for locale(s) to be generated (UTF-8 is the recommend choice).

**Select locale**
Example for a system in Denmark using english messages
```bash
...
en_US.UTF-8 UTF-8
...
```
**Generate the messages**
```bash
# locale-gen
```

**locale.conf**
Edit your locale configuration in **/etc/locale.conf** to match above choice - example for Denmark
```bash
LANG=en_DK.UTF8
```

### Timezone
Set the time zone for location (the available zones is listed in **/usr/share/zoneinfo/** using the **Continent/Capitol** format).

Symlink the time zone as **/etc/localtime** - example for Denmark
```bash
# ln -sf /usr/share/zoneinfo/Europe/Copenhagen /etc/localtime
```

### Clock
Linux clock runs using the timezone info and UTC time.
```bash
# hwclock --systohc --utc
```

### Hostname

Set hostname
```bash
# echo manjaro > /etc/hostname
```

### Hosts configuration
>```
># nano /etc/hosts
>```
>---
>```bash
>127.0.0.1    localhost
>::1          localhost
>127.0.1.1    manjaro.localdomain manjaro
>```
Note: If the system has a static IP replace 127.0.1.1 with the IP.

### System administration
Allow members of the *wheel* group to perform administrative tasks.

Run *visudo*
```bash
# visudo
```
Locate the line reading *# %wheel ALL=(ALL) ALL* and remove the **#** in the beginning of the line
```bash
%wheel ALL=(ALL) ALL
```
Enable dhcp network
```bash
# systemctl enable dhcpcd
```
Enable ssh service
```bash
# systemctl enable sshd
```

### Time syncronization
Enable timesync daemon
```bash
# systemctl enable systemd-timesyncd
```

### Root password
Set a password the root user
```bash
# passwd
```

### cloud-init
```bash
# pacman -S cloud-init
```
Edit `/etc/cloud/cloud.cfg` add the line

```text
datasource_list: [ ConfigDrive, OpenNebula, DigitalOcean, Azure, AltCloud, OVF, MAAS, GCE, OpenStack, CloudSigma, SmartOS, None, NoCloud  ]
```

In the section **cloud_init_modules** remember to remove the line specifying `growpart` unless you have build and installed the **growpart** package from AUR.

Edit the section **system_info** - edit the section as preferred.

Enable the following services

```bash
# systemctl enable cloud-init.service cloud-final.service
```

## 8. Bootloader
---
Build the initramfs and install and setup grub

### Initramfs
Build the initramfs according to your chosen kernel e.g. *linux54*
```bash
# mkinitcpio -p linux54
```

### Bootloader

```bash
# grub-install --target=i386-pc /dev/sda
```
Edit the default grub configuration `/etc/default/grub` and adjust to match your setup e.g. you can use the label or you can use device path for your root partition
```
GRUB_CMDLINE_LINUX_DEFAULT=".. root=/dev/sdaX ..."
```
or
```
GRUB_CMDLINE_LINUX_DEFAULT=".. root=LABEL=MJROROOT ..."
```
and uncomment the following line
```
#GRUB_DISABLE_LINUX_UUID=true
```

Generate grub configuration

```bash
# grub-mkconfig -o /boot/grub/grub.cfg
```

### Close chroot
```bash
# exit
```
### Unmount the partitions
```bash
# umount -R /mnt
```

### Testing the image is booting
After you have booted your image - check the folder `/run/cloud-init` for non-trivial messages.
Remember to eject the ISO you used for installation
Clean your image and the cloud-init before shutting down

```bash
# pacman -Scc
# cloud-init clean
# systemctl poweroff
```

Then compress the VDI image - best result
```bash
# bzip2 --best -k your-vm-disk.vdi
```