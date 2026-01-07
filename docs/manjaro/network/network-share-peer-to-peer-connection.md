---
title: 'Share network connection peer-to-peer'
date: '13:43 31-12-2023'
taxonomy:
    category:
        - docs
---


Computer A - having internet connection- using phone tether or wlan
```
nmcli connection modify myconnection ipv4.method shared
sudo systemctl restart NetworkManager
```

Computer B having no network
```
nmcli connection modify myconnection ipv4.method auto
sudo systemctl restart NetworkManager
```
Connect ethernet cable

Source [Connect 2 PCs, one has internet via USB-tethering - Manjaro Linux Forum][1]

[1]: https://forum.manjaro.org/t/connect-2-pcs-one-has-internet-via-usb-tethering/154208/6

