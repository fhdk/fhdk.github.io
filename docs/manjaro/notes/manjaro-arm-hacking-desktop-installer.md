---
title: 'Hacking ARM desktop installer'
date: '07:22 12-10-2022'
taxonomy:
    category:
        - docs
---

Working around the desktop installer image requiring peripherals to complete.

Prerequisite

* network scanner e.g. **nmap**, **arp-scan** or **ipscan**
* ethernet connection

Raspberry Pi connected using ethernet

1. Download desktop the image
2. Write image to sd-card
3. Insert image into your RPI
4. Connect ethernet cable
5. Power device
6. Wait until the device settles - the green activity led becomes quiet
7. Scan the network and locate the device IP
    ```
    nmap -p 22 --open ip.x.y.0/24
    ```
8. Open an ssh connection to the device using **oem** as username and password
    ```
    ssh oem@ip.x.y.z
    ```
9. Ensure the system is up-to-date with a current mirror and current metadata
   ```
   sudo pacman-mirrors --continent && sudo pacman -Syy
   ```
10. Remove the calamares oem installer
    ```
    sudo pacman -Rns calamares calamares-arm-oem
    ```
11. Sync the command line oem installer
    ```
    sudo pacman -S manjaro-arm-oem-install
    ```
12. Execute the oem setup script
    ```
    sudo bash /usr/share/manjaro-arm-oem-install/manjaro-arm-oem-install
    ```
13. Done

## Remote GUI
A headless system with a GUI desktop is useless unless you can access that desktop from elsewhere.

[https://root.nix.dk/network-notes/tigervnc-over-ssh][2]

Cross posted to [Manjaro Forum][1]

[1]: https://forum.manjaro.org/t/root-tip-how-to-hacking-the-arm-desktop-installer/124048
[2]: https://root.nix.dk/network-notes/tigervnc-over-ssh