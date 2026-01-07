---
title: 'Pinebook Pro Network Wifi Privacy Cycle'
date: '13:41 13-08-2023'
taxonomy:
    category:
        - docs
---

All credit - @DrYak at https://forum.pine64.org/showthread.php?tid=8313&pid=52645#pid52645

update 2020.06.16: some recent kernel update has changed the naming of the wifi device - now fe310000.mmc. For those who like a compact one-liner:

```
sudo tee /sys/bus/platform/drivers/dwmmc_rockchip/{un,}bind <<< "$(basename /sys/devices/platform/fe310*)"
```

TL;DR: use the two commands here to restart the Wifi driver after powering the Wifi again with the Privacy Switch for Wifi (Pine + F11) :
```
$ echo 'fe310000.dwmmc' | sudo tee /sys/bus/platform/drivers/dwmmc_rockchip/unbind
$ echo 'fe310000.dwmmc' | sudo tee /sys/bus/platform/drivers/dwmmc_rockchip/bind
```

The full story :

(Moving to a more general thread, because currently my reply is burried in the middle of the Manjaro discussion)

Currently, the privacy switch for Wifi cannot put back Wifi on because soldered-on-motherboard SDIO modules aren't Plug'n'Play.

Here's what you normally see if you try the usual rmmod  + modprobe route:

```
$ sudo rmmod brcmfmac brcmutil cfg80211
$ modprobe brcmfmac
$ dmesg
[ 1299.511038] usbcore: deregistering interface driver brcmfmac
[ 1313.114017] platform regulatory.0: Direct firmware load for regulatory.db failed with error -2
[ 1313.114781] cfg80211: failed to load regulatory.db
[ 1313.132537] brcmfmac: probe of mmc0:0001:1 failed with error -110
[ 1313.133192] brcmfmac: probe of mmc0:0001:2 failed with error -110
[ 1313.133865] usbcore: registered new interface driver brcmfmac
```

There is a way though:

You can find the actual soldered device that answers on the mmcn mentioned above and unbind and re-bind it.

```
$ l`s -d /sys/devices/platform/*.dwmmc/mmc_host/mmc0
/sys/devices/platform/fe310000.dwmmc/mmc_host/mmc0
$ ls -d /sys/devices/platform/*.dwmmc/mmc_host/mmc*/mmc*/mmc*/ieee80211
/sys/devices/platform/fe310000.dwmmc/mmc_host/mmc0/mmc0:0001/mmc0:0001:2/ieee80211
$ echo 'fe310000.dwmmc' | sudo tee /sys/bus/platform/drivers/dwmmc_rockchip/unbind
$ echo 'fe310000.dwmmc' | sudo tee /sys/bus/platform/drivers/dwmmc_rockchip/bind
$ dmesg
[ 1845.593897] mmc0: card 0001 removed
[ 1849.712047] dwmmc_rockchip fe310000.dwmmc: IDMAC supports 32-bit address mode.
[ 1849.712707] dwmmc_rockchip fe310000.dwmmc: Using internal DMA controller.
[ 1849.713307] dwmmc_rockchip fe310000.dwmmc: Version ID is 270a
[ 1849.713840] dwmmc_rockchip fe310000.dwmmc: DW MMC controller at irq 27,32 bit host data width,256 deep fifo
[ 1849.714799] dwmmc_rockchip fe310000.dwmmc: allocated mmc-pwrseq
[ 1849.715321] mmc_host mmc0: card is non-removable.
[ 1849.728861] mmc_host mmc0: Bus speed (slot 0) = 400000Hz (slot req 400000Hz, actual 400000HZ div = 0)
[ 1849.777673] mmc_host mmc0: Bus speed (slot 0) = 148500000Hz (slot req 150000000Hz, actual 148500000HZ div = 0)
[ 1850.622467] dwmmc_rockchip fe310000.dwmmc: Successfully tuned phase to 47
[ 1850.627007] mmc0: new ultra high speed SDR104 SDIO card at address 0001
[ 1850.636215] brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac43456-sdio for chip BCM4345/9
[ 1852.731997] brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac43456-sdio for chip BCM4345/9
[ 1852.732808] brcmfmac: brcmf_c_process_clm_blob: no clm_blob available (err=-2), device may have limited channels available
[ 1852.734049] brcmfmac: brcmf_c_preinit_dcmds: Firmware: BCM4345/9 wl0: Sep  7 2018 14:33:37 version 7.45.96.27 (42b546f@shgit) (r) FWID 01-c958c084 es7.c5.n4.a3
[ 1860.053215] IPv6: ADDRCONF(NETDEV_CHANGE): wlan0: link becomes ready
```

Note if I'm not mistaken :
> the 'mmc0' numbering might change depending on how your OS enumerate its devices (see how /dev/mmcblk changes - on Manjaro the Wifi is on mmc0 and the internal eMMC is on mmc2, but on the factory shipped Debian - Wifi is on mmc2 and mmc0 has the external µSD)
> but fe310000.dwmmc is how this soldered MMC interface is mapped to the Rockchip and is fixed accross OSes (it is set in the .dtb)


the bind + unbind trick can also be used if the Broadcom Wifi chip has gone crazy and needs restarting.

(source: other people with similar problems of soldered MMC modules) 

