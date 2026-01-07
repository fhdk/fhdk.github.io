---
title: 'Intel Xorg config'
taxonomy:
    category:
        - docs
date: '07:58 13-01-2020'
---

## Make the best of Intel
Install the mesa package.
```text
pacman -Syu mesa
```
### Enable early kms
Edit **/etc/mkinitcpio.conf**
```text
MODULES=(i915)
```
Then build initramfs
```text
mkinitcpio -P
```
### Xorg configuration
Edit **/etc/X11/xorg.conf.d/20-intel.conf**
Option 1
```text
Section "Device"
	Identifier  "Intel Graphics"
	Driver      "intel"
	Option      "TearFree" "true"
EndSection
```
Option 2
```text
Section "Device"
	Identifier  "Intel Graphics"
	Driver      "intel"
    Option      "AccelMethod"     "sna"
	Option      "TearFree"        "true"
    Option      "SwapbuffersWait" "true"
    Option      "DRI"             "2"
EndSection
```
```
Reboot
```
