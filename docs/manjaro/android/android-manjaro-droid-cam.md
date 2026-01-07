---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Droid Cam on Manjaro'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## Droidcam

- **[Droidcam][1]** web
- Download  **[GNU/Linux client][7]**

> A tool for using your android device as a wireless/usb webcam

## Problems
Judging from the a [forum search][2] for droidcam it appears Manjaro users are having a difficult time getting this into a working state.

So I decided to check if it is such a pain to get working.

## Droidcam on AUR
Despite some comments on the AUR page on non-versioned package source it seems the maintainer and the droidcam developers has decided it is a good idea to version the package source - probably taking into account when an AUR helper is caching the source archive.

The comments track is filled with issues not only with relation to the build but also sound issues.

The maintainer also states that [Manjaro is not supported][6].

We will take the sound issues later - first we take a look at getting your android device camera connected.

## Your Android device
Before you even think of getting your computer to connect you must install the DroidCam application on your android device.

My device is a rooted Huawei Nexus 6P using a crDroid Android 10 without any Google related stuff (and I mean nothing at all).

There is several possible ways to get hold of the apk - I downloaded the apk, renamed it and installed it using adb debug

    adb install droidcam-6.7.10.apk

It is necessary to setup the DroidCam and to start the App to be able to connect to it from your computer.

## Your Manjaro system
My Manjaro system is on unstable branch and using kernel 5.7.4 using Openbox window manager.

The maintainer of the PKGBUILD states that [only **makepkg** is supported][4] - so let's use it to build the package.

Update your system and install the necessary build tools

    sudo pacman -Syu --needed git base-devel

Next thing is to get hold of the PKGBUILD. Open a terminal and clone the repo

```
➜  ~ git clone https://aur.archlinux.org/droidcam              
Cloning into 'droidcam'...
remote: Enumerating objects: 110, done.
remote: Counting objects: 100% (110/110), done.
remote: Compressing objects: 100% (98/98), done.
remote: Total 110 (delta 11), reused 109 (delta 11), pack-reused 0
Receiving objects: 100% (110/110), 43.37 KiB | 2.55 MiB/s, done.
Resolving deltas: 100% (11/11), done.
```

Navigate into the folder and list the content

```
➜  ~ cd droidcam                                
➜  droidcam git:(master) ls
droidcam.conf  droidcam.desktop  PKGBUILD
```
What we are interested in is the PKGBUILD as it is a good habit to verify the content

[details="droidcam/PKGBUILD"]
```
➜  droidcam git:(master) cat PKGBUILD 
# Maintainer: AwesomeHaircut <jesusbalbastro at gmail com>
# Maintainer: Mateusz Gozdek <mgozdekof@gmail.com>
# Contributor: Rein Fernhout <public@reinfernhout.xyz>
# Past Contributor: James An <james@jamesan.ca>

pkgname=droidcam
pkgver=1.3
pkgrel=1
epoch=1
pkgdesc='A tool for using your android device as a wireless/usb webcam'
arch=('x86_64')
url="https://github.com/aramg/${pkgname}"
license=('GPL')
depends=('v4l2loopback-dc-dkms' 'alsa-lib' 'libjpeg-turbo' 'ffmpeg')
makedepends=('gtk3')
optdepends=('gtk3: use GUI version in addition to CLI interface' )

source=("${pkgname}.desktop"
        "${pkgname}-${pkgver}.zip::${url}/archive/v${pkgver}.zip"
        "${pkgname}.conf"
)

sha512sums=('72d21aa2d7eecc9bb070aaf7059a671246feb22f9c39b934a5463a4839f9347050de00754e5031dbc44f78eb2731f58f0cd2fcf781bc241f6fbd1abb4308b7ee'
            'c783b62c530c521aa7f047073efe74b57f28fbadbd097dca595fb582820566aedd03c6e92d2f24d9ff84dceed8ab51955ad77e80481ebfb6e30423425f8f2953'
            'ea457b46a2fc9f1a3ea8e99f2cd0771a587cff89f42335fdaf55988dda0376a1fea73b660174c9f1906a304bace68bffec30b70b20dafc05ebae8854d9aadb13')

build() {
  cd ${pkgname}-${pkgver}/linux

  # All JPEG* parameters are needed to use shared version of libturbojpeg instead of
  # static one.
  make JPEG_DIR="" JPEG_INCLUDE="" JPEG_LIB="" JPEG=$(pkg-config --libs --cflags libturbojpeg)
}

package() {
  pushd ${pkgname}-${pkgver}/linux

  # Install droidcam program files
  install -Dm755 "${pkgname}" "${pkgdir}/usr/bin/${pkgname}"
  install -Dm755 "${pkgname}-cli" "${pkgdir}/usr/bin/${pkgname}-cli"
  install -Dm644 icon2.png "${pkgdir}/usr/share/pixmaps/${pkgname}.png"
  install -Dm644 "../../${pkgname}.desktop" "${pkgdir}/usr/share/applications/${pkgname}.desktop"
  install -Dm644 "../../${pkgname}.conf" "${pkgdir}/etc/modules-load.d/droidcam.conf"
  install -Dm644 README.md "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
```
[/details]

We find the expected stuff - a list of dependencies - and note that we have a **dkms** dependecy  - and from the **package()** function it is worth noting that we get a new configuration loading the **v4l** modules required to connect to a live camera.

The dkms dependency is tricky because it usually requires kernel headers to be installed for your kernel. While Arch only has clearly distinctive kernels with only one version - Manjaro has several versions and every version has a complimentary header package.

So to be able to build droidcam you need to ensure the headers are installed for all of your kernels - otherwise the droidcam app will not build and ultimately not work.

List your installed kernels - example from my system
```
➜  droidcam git:(master) mhwd-kernel -li  
Currently running: 5.7.4-1-MANJARO (linux57)
The following kernels are installed in your system:
   * linux56
   * linux57
```
With this info - please use the output from your system - install the kernel headers 

    sudo pacman -Syu linux56-headers linux57-headers

Now we are ready to build and install the package and the dependencies. When the build is done - you will be asked to authorize the package installation.

[details="makepkg -is"]
```
➜  droidcam git:(master) makepkg -is
==> Making package: droidcam 1:1.3-1 (fre 19 jun 2020 12:36:46 CEST)
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
  -> Found droidcam.desktop
  -> Downloading droidcam-1.3.zip...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   117  100   117    0     0   1218      0 --:--:-- --:--:-- --:--:--  1218
100 91207    0 91207    0     0   126k      0 --:--:-- --:--:-- --:--:--  126k
  -> Found droidcam.conf
==> Validating source files with sha512sums...
    droidcam.desktop ... Passed
    droidcam-1.3.zip ... Passed
    droidcam.conf ... Passed
==> Extracting sources...
  -> Extracting droidcam-1.3.zip with bsdtar
==> Starting build()...
gcc -Wall -no-pie src/droidcam-cli.c src/connection.c src/settings.c src/decoder.c src/decoder_snd.c src/decoder_v4l2.c src/av.c src/adb.c -lturbojpeg `pkg-config --libs --cflags libswscale libavutil` -lspeex -lasound -lpthread -lm -o droidcam-cli
gcc -Wall -no-pie src/droidcam.c src/resources.c src/connection.c src/settings.c src/decoder.c src/decoder_snd.c src/decoder_v4l2.c src/av.c src/adb.c `pkg-config --libs --cflags gtk+-3.0` `pkg-config --libs x11` -lturbojpeg `pkg-config --libs --cflags libswscale libavutil` -lspeex -lasound -lpthread -lm -o droidcam
src/droidcam.c: In function ‘the_callback’:
src/droidcam.c:189:4: warning: ‘gtk_menu_popup’ is deprecated: Use '(gtk_menu_popup_at_widget, gtk_menu_popup_at_pointer, gtk_menu_popup_at_rect)' instead [-Wdeprecated-declarations]
  189 |    gtk_menu_popup(GTK_MENU(menu), NULL, NULL, NULL, NULL, 0, 0);
      |    ^~~~~~~~~~~~~~
In file included from /usr/include/gtk-3.0/gtk/gtklabel.h:34,
                 from /usr/include/gtk-3.0/gtk/gtkaccellabel.h:35,
                 from /usr/include/gtk-3.0/gtk/gtk.h:33,
                 from src/droidcam.c:11:
/usr/include/gtk-3.0/gtk/gtkmenu.h:138:9: note: declared here
  138 | void    gtk_menu_popup    (GtkMenu        *menu,
      |         ^~~~~~~~~~~~~~
==> Entering fakeroot environment...
==> Starting package()...
~/droidcam/src/droidcam-1.3/linux ~/droidcam/src
==> Tidying install...
  -> Removing libtool files...
  -> Purging unwanted files...
  -> Removing static library files...
  -> Stripping unneeded symbols from binaries and libraries...
  -> Compressing man and info pages...
==> Checking for packaging issues...
==> Creating package "droidcam"...
  -> Generating .PKGINFO file...
  -> Generating .BUILDINFO file...
  -> Generating .MTREE file...
  -> Compressing package...
==> Leaving fakeroot environment.
==> Finished making: droidcam 1:1.3-1 (fre 19 jun 2020 12:36:48 CEST)
==> Installing package droidcam with pacman -U...
loading packages...
resolving dependencies...
looking for conflicting packages...

Packages (1) droidcam-1:1.3-1

Total Installed Size:  0,12 MiB

:: Proceed with installation? [Y/n] 
(1/1) checking keys in keyring                     [#######################] 100%
(1/1) checking package integrity                   [#######################] 100%
(1/1) loading package files                        [#######################] 100%
(1/1) checking for file conflicts                  [#######################] 100%
(1/1) checking available disk space                [#######################] 100%
:: Processing package changes...
(1/1) installing droidcam                          [#######################] 100%
Optional dependencies for droidcam
    gtk3: use GUI version in addition to CLI interface [installed]
:: Running post-transaction hooks...
(1/2) Arming ConditionNeedsUpdate...
(2/2) Updating the desktop file MIME type cache...
```
[/details]

You should get a similar output and if you don't you have inconsistencies within your system.

I can't say what it might be - but suffice to say - as the time of writing this guide - the droidcam package builds without hickups

## Starting the DroidCam app
The app is depending on the correct v4l modules to be loaded by the kernel. If the kernel is updated but the system not restarted - there will be a mismatch between the kernel and modules which will make the application fail.

Better safe than sorry - reboot your system - or manually load the required modules which are described in droidcam.conf. Manually loading the modules do not guarantee droidcam will work - your best shot is still a restart of your system.

```
➜  droidcam git:(master) cat droidcam.conf 
videodev
v4l2loopback-dc
➜  droidcam git:(master) sudo modprobe videodev v4l2loopback-dc
```

### WiFi/LAN
To connect using WiFi you must ensure your phone is on the same local network as your computer. When you start DroidCam on your device it will display the IP address to connect to.

Depending on what you want to do

Connect using a browser
* **`http://<phoneip>:4747/`**
* **`http://<phoneip>:4747/video`**

Using the DroidCam app


[details="Screenshot"]
![image|664x496](upload://dvt5wwExY78eZl61Qo5wuKhQdbO.jpeg) 
[/details]


## Sound
Making it possible to use sound too - you are required set the properties in the pulseaudio mixer. I have not invested time in getting this to work.

As of [date=2020-06-02 time=14:13:00 timezone="Europe/Copenhagen"] sound support is an [ongoing issue][3].

Another [comment][5] in the thread mentions an **install-sound** script?

## Forum issues list

* https://archived.forum.manjaro.org/t/droidcam-doesnt-work/148707?u=linux-aarhus
* https://archived.forum.manjaro.org/t/package-v4l2loopback-dc-dkms-needed-for-droidcam-fails-to-compile-with-both-dkms-and-make/146132?u=linux-aarhus
* https://archived.forum.manjaro.org/t/manjaro-is-incompatible-with-an-arch-linux-pkgbuild/145499?u=linux-aarhus
* https://archived.forum.manjaro.org/t/droidcam-6-7-5-5-error-extra-modules-exist-while-building-with-yay/142345?u=linux-aarhus
* https://archived.forum.manjaro.org/t/anybody-using-droidcam-for-a-cellphone-webcam/131821?u=linux-aarhus
* https://archived.forum.manjaro.org/t/droidcam/119211?u=linux-aarhus
* https://archived.forum.manjaro.org/t/how-do-i-enable-phone-as-webcam-for-school/130143
* https://archived.forum.manjaro.org/t/couldnt-install-droidcam-using-yay/80837?u=linux-aarhus

[1]: https://www.dev47apps.com/
[2]: https://archived.forum.manjaro.org/search?q=droidcam
[3]: https://aur.archlinux.org/packages/droidcam/#comment-748930
[4]: https://aur.archlinux.org/packages/droidcam/?O=10&PP=10#comment-748354
[5]: https://aur.archlinux.org/packages/droidcam/?O=20&PP=10#comment-748169
[6]: https://aur.archlinux.org/packages/droidcam/?O=50&PP=10#comment-737251
[7]: https://www.dev47apps.com/droidcam/linuxx/
