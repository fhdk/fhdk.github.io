---
title: 'Flash USG-3P'
date: '11:28 30-03-2023'
taxonomy:
    category:
        - docs
---

## Unifi Security Gateway (USG-3)

Flash using console port with usb-to-serial rollover console cable and picocom

* Download the orginal complete firmware image, version 4.2: https://dl.ubnt-ut.com/cmb/USG-4_2_0-shipped.img.bz2
* Extract the img file from the archive.
* Unplug the USG and remove the rubber feet from the USG.
* Unscrew the screws beneath the rubber feet. 
* Remove the top plate to access the inside (gently pry the top off if necessary).
* Gently remove the USB drive and insert it into your computer.
* Use dd to write img to usb.
* Insert the USB drive into the USG.

Connect the console cable and start minicom and power up the USG

    picocom -b 115200

You will be prompted for username/password which is **ubnt:ubnt**

Now you need to tell the USG to update with software from a web address. Find the latest USG firmware from Unifi here or elsewhere on the Unifi site: https://www.ui.com/download/unifi-switching-routing/default/default/. 

Click the download button and agree to the terms - then copy the download link e.g. https://dl.ui.com/unifi/firmware/UGW3/4.4.57.5578372/UGW3.v4.4.57.5578372.tar.

In picocom, enter: 

```
upgrade https://dl.ui.com/unifi/firmware/UGW3/4.4.57.5578372/UGW3.v4.4.57.5578372.tar
```

If don't get any response leave let the device be for 5 or 10 minutes. The USG will reboot when finished updating.

References:
* https://www.reddit.com/r/Ubiquiti/comments/kw4bx3/how_to_flash_and_recover_unifi_security_gateway/
* https://support.hostifi.com/en/articles/6629721-unifi-how-to-fix-a-corrupted-usg-3p