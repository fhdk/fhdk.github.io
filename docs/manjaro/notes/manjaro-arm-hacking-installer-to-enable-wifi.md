---
title: 'Hacking ARM install image to connect to WiFi'
date: '10:01 12-10-2022'
taxonomy:
    category:
        - docs
---

**Difficulty: ★★☆☆☆**

## ARM image WiFi connection hack

This is specifically useful for Pi Zero W but it works for any Pi with onboard wifi.

Assuming you already have downloaded a Manjaro ARM image (any flavor) and have the urge to connect over WiFi to finalize the setup process.

Open a terminal and do

```text
su -l root
```

Mount the root image from the sdcard

```text
mkdir tmproot
mount /dev/mmcblk0p2 tmproot
```

Create a network definition

```text
cat << EOF >> root/etc/systemd/network/wlan0.network
[Match]
Name=wlan0

[Network]
DHCP=yes
EOF
```

Create a wpa_supplicant configuration - replace the variables **$SSID** and **$PASS** with your actual values

```text
wpa_passphrase "$SSID" "$PASS" > tmproot/etc/wpa_supplicant/wpa_supplicant-wlan0.conf
```

Create a symlink to start wpa_supplicant for the wlan0

```text
ln -s /usr/lib/systemd/system/wpa_supplicant@.service \
      tmproot/etc/systemd/system/multi-user.target.wants/wpa_supplicant@wlan0.service
```

Unmount the partition and exit root context

```text
umount tmproot
rm -r tmproot
exit
```
Insert the sd-card in your pi and power up

[https://root.nix.dk/en/utility-scripts/manjaro-arm-image-wifi-hack](https://root.nix.dk/en/utility-scripts/manjaro-arm-image-wifi-hack)
[https://root.nix.dk/en/manjaro/hacking-arm-desktop-installer](https://root.nix.dk/en/manjaro/hacking-arm-desktop-installer)

Cross posted at [Manjaro Forum][1]

[1]: https://forum.manjaro.org/t/root-tip-how-to-hack-arm-install-image-to-connect-to-your-wifi/124052