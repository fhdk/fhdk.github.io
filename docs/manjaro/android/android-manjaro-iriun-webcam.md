---
published: true
date: '24-10-2019 11:00'
publish_date: '24-10-2019 11:00'
title: 'Iriun Web Cam on Manjaro'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Iriun Web Cam

- **[Iriun][1]** web cam

> A tool for using your android device as a wireless/usb webcam and VR

## Your Android device
Before you even think of getting your computer to connect you must install the [Iriun 4K Webcam][2] application on your android device.

## Your Manjaro system
Manjaro and Arch are not identical systems and Manjaro kernel naming will require you to pre install the kernel headers for your installed kernels. If you fail to do so - the required `v4l2loopback-dkms` package will fail to build the required kernel modules and you will not be able to connect to phone.

### Prepare system to build packages
Update your system and install the necessary build tools

    sudo pacman -Syu --needed git base-devel

### Clone the PKGBUILD sources
Next thing is to get hold of the PKGBUILD. Open a terminal and clone the repo

```
$ git clone https://aur.archlinux.org/iriunwebcam-bin.git
Cloning into 'iriunwebcam-bin'...
....
Resolving deltas: 100% (11/11), done.
```

### Inspect the sources
Navigate into the folder and list the content
```
$ cd iriunwebcam-bin
$ ls -A
.git  PKGBUILD  .SRCINFO
```

We are interested in the content of the PKGBUILD (and possible companion files) because it is best practise to verify the content

As of __2022-09-04__ the content is as follows
```
$ cat PKGBUILD 
# Maintainer: Barfin
# Contributor: DanielH, agstrc

pkgname=iriunwebcam-bin
pkgver=2.7
pkgrel=1
pkgdesc="Use your phone's camera as a wireless webcam in your PC."
arch=('any')
url="http://iriun.gitlab.io"
license=(unknown)
source=("iriunwebcam-${pkgver}.deb::http://iriun.gitlab.io/iriunwebcam-${pkgver}.deb")
options=('!strip')
depends=('jack' 'qt5-base' 'v4l2loopback-dkms' 'android-tools')

package() {
    tar -xf "${srcdir}/data.tar.xz" -C "${pkgdir}"

    # The permissions on the upstream file are wrong (eg: /etc is set to 775 instead of 755)
    # The command below sets directories to 755 but files to 644
    chmod -R u+rwX,go+rX,go-w "${pkgdir}"
    chmod 755 "${pkgdir}/usr/local/bin/iriunwebcam"
}

md5sums=('0c96650189ad6f579e6104c62e3b50c4')
```

We find the expected stuff - a list of dependencies

* jack (audio)
* qt5-base
* v4l2loopback-dkms
* android-tools

:exclamation:  Note the **dkms** dependency (v4l2loopback-dkms)
:exclamation: Note the **android-tools** dependency

### More preparation according to PKGBUILD content
#### Android tools
The android-tools dependency creates a new group __adbusers__. To be able to connect using adb you must add your username to _adbusers_ group

    sudo gpasswd -a $USER adbusers

#### DKMS and kernel headers
The dkms dependency is tricky because it usually requires kernel headers to be installed for your kernel. While Arch only has a clear distinctive kernel naming with only one version - Manjaro has several versions and every version has a complimentary header package.

So to be able to build the package you need to ensure the headers are installed for all of your kernels - otherwise the app will not build and ultimately not work.

List your installed kernels - your output may differ
```bash
$ mhwd-kernel -li         
Currently running: 5.9.1-3-MANJARO (linux59)
The following kernels are installed in your system:
   * linux515
```
With this info - replace $KERNEL with the values from your system - install the kernel headers for the kernel(s) mentioned above

```bash
sudo pacman -Syu $KERNEL-headers
```

###  Ensure correct checksums
If the sources has changed and the maintainer has not yet updated the PKGBUILD you need to do it manually.

You can do the using the __updpkgsums__ command or you can add the __-g__ argument to makepkg to this at build time
```
$ updpkgsums
```

### Build the package
Now we are ready to build the package and install the dependencies - optionally adding the argument to recreate the integrety checks

```
$ makepkg -sf [-g]
...

==> Finished making: iriunwebcam-bin 2.7-1 (2022-09-04T11:18:38 CEST)
```

You should get a similar result. The resulting package will likely be a different version.

When the build is done - inspect the build folder once more. If you're paranoid, superuser or developer - you may inspect the package before installation(Midnight Commander is an excellent tool). 

When examining the pacage `iriunwebcam-bin-$VERSION-any.pkg.tar.zst` we find the package to contain configurations
```
$ cat /etc/modprobe.d/iriunwebcam-options.conf
#
options v42loopback exclusive_caps=1 devices= 1 card_label="Iriun Webcam"
```
```
$ cat /etc/modules-load.d/iriunwebcam.conf
#
v4l2loopback
```
A binary
```
/usr/local/bin/iriunwebcam
```
A .desktop launcher
```
/usr/share/applications/iriunwebcam.desktop
```
And a pixmap
```
/usr/share/pixmaps/iriunwebcam.png
```

### Install the resulting packages
To actually install the package use pacman - replace $VERSION with the version you have just built.
```bash
sudo pacman -U iriunwebcam-bin-$VERSION-any.pkg.tar.zst
```
Reboot your system to have the system use the new modules.

## Using your phones camera
Follow the developer's instruction to connect your camera.

* Connect your phone to the same wifi network as your computer 
* You can also connect using USB.
   - if you use usb then activate developer mode then scroll down and activate usb-debugging
   - usb debugging requires confirmation to trust the computer
* Start the app on your computer
* Start the app on your phone

As noted above - I don't know why but it is verified - if your computer is wired to the network - you need to use USB to be able to use your phones camera - it is possible it is a limitation within the software as droid-cam will work no matter the computers connection is wired or wireless.

## Changelog
* 2022-09-04 - Updated with new dependency and paragraphing to clarify the elements of the guide.

[1]: https://iriun.com
[2]: https://play.google.com/store/apps/details?id=com.jacksoftw.webcam