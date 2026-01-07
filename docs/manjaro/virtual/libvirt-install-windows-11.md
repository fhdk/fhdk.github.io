---
title: 'virt-manager - Windows 11'
date: '07:53 30-10-2024'
taxonomy:
    category:
        - docs
---

## Install Win11 using libvirt

- Create a new virtual machine
- Select a Windows 11 ISO file
- Accept the defaults clicking **Next** until you reach the final screen
- Tick the box **Customize configuration before install** and click **Finish**
- In the Overview pane - set vm firmware to **BIOS** and click Apply
- Click the button Begin Installation

## Installer
Before starting the installer press <kbd>Shift</kbd><kbd>F10</kbd> to launch the Windows Cmd utility then launch **regedit**.

- Expand HKEY_LOCAL_MACHINE\SYSTEM\Setup
- Add new key named **LabConfig**
- Add new DWORD value with name BypassTPMCheck and change the value to 1.
- Add new DWORD value with name BypassRAMCheck and change the value to 1.
- Add new DWORD value with name BypassSecureBootCheck and change the value to 1.

Close the registry editor and exit the shell, then continue the installer

## Bypass network check

During last stage the installer will insist in network access.

This can be disabled using the Cmd utility <kbd>Shift</kbd><kbd>F10</kbd> 

- enter **OOBE\BYPASSNRO** and press <kbd>Enter</kbd>
- close the Cmd utility
- back in the setup window click **I don't have internet**
