---
title: 'macOS installer'
taxonomy:
    category:
        - docs
---

## To download a full installer

For macOS Big Sur
```text
softwareupdate --fetch-full-installer --full-installer-version 11.3
```
Replacing the version will download the version - e.g. Catalina has version 10.15.3

## Converting installer to iso

```
hdiutil create -o /tmp/BigSur -size 12500m -volname BigSur -layout SPUD -fs HFS+J
```
```
hdiutil attach /tmp/BigSur.dmg -noverify -mountpoint /Volumes/BigSur
```
```
sudo /Applications/Install\ macOS\ Big\ Sur/Contents/Resources/createinstallmedia --volume /Volumes/BigSur --nointeraction
```
```
hdiutil detach /Volumes/BigSur/
```
```
hdiutil convert /tmp/BigSur.dmg -format UDTO -o ~/Desktop/BigSur.cdr
```

```
mv ~/Desktop/BigSur.cdr.dmg ~/Desktop/BigSur.iso
```

## Create Virtualbox VM

```
VBoxInternal/Devices/efi/0/Config/DmiSystemProduct" "MacBookPro11,3"
```
```
VBoxManage setextradata "Big Sur" "VBoxInternal/Devices/efi/0/Config/DmiSystemVersion" "1.0"
```
```	
VBoxManage setextradata "Big Sur" "VBoxInternal/Devices/efi/0/Config/DmiBoardProduct" "Mac-2BD1B31983FE1663"
```
```
VBoxManage setextradata "Big Sur" "VBoxInternal/Devices/smc/0/Config/DeviceKey" "ourhardworkbythesewordsguardedpleasedontsteal(c)AppleComputerInc"
```
```
VBoxManage setextradata "Big Sur" "VBoxInternal/Devices/smc/0/Config/GetKeyFromRealSMC" 1
```

## VM screen size
It seems macOS on VBox is limited to 1024x768, 1280x720, 1920x1080, 2560x1440
```
VBoxManage setextradata "Big Sur" "VBoxInternal2/EfiGraphicsResolution 1280x720"
```