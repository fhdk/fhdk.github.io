---
title: Install with local account
date: 07:26 11-10-2025
taxonomy:
  category:
    - docs
permissions:
  inherit: true
  authors:
    - linux-aahus
---

The obvious workaround is - if you can choose -the choose Linux, Debian, Arch Linux, Manjaro, Ubuntu, Linux Mint .. the shelf is big.


## Workaround 1 [[1]]
When prompted to create a Microsoft account, enter the current date as your date of birth. This will cause the creation of a Microsoft account to be rejected because the US COPPA (Children's Online Privacy Protection Act) prohibits children from creating accounts.

Microsoft is unlikely to be able to disable this workaround for legal reasons. Although an error message appears during setup, the option to create a local account is offered, writes Bolko.

## Workaround 2 (for Windows 11 Home) [[1]]

During the OOBE phase of setup, press the key combination Ctrl + Shift + F3 during the account creation step to enter audit mode. Then type the following command into the command prompt:

    slui.exe /upk changepk.exe /ProductKey

By switching from Home to Generic-Pro, there should now be a new option to join a domain, which allows you to create a local account. Later, after installing Windows 11, you can change the product key back to Home, but the local account will remain.

## Workaround 3 [[1]]

An autounattend.xml response file is used to create a local account. I discussed such files in the article Windows 11 24H2: Security issue caused by unattend.xml.

## Workaround 4 [[1]]

During the OOBE phase, when creating the user account, you can press the key combination Shift + F10 and enter the following command in the command prompt.

    WinJS.Application.restart("ms-cxh://localonly")

## Workaround 5 [[2]]

During OOBE

- Disconnect your internet connection
- <kbd>Shift</kbd><kbd>F10</kbd>
```
reg edd HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\OOBE /v BypassRRO /t REG_DWORD /d 1 /f shutdown /r /t 0
```

## Workaround 6 [[2]]

During OOBE

- <kbd>Shift</kbd><kbd>F10</kbd>
```
net user "User Name" /add
net localgroup "Adminitrators" "User Name" /add
cd OOBE
msoobe && shutdown -r
```

## Workaround 7

Keep an earlier version of the Windows installation ISO around - upgrade to the latest version.

## Source
- Windows 11 setup: Microsoft will block the creation of local accounts in the future (borncity.com) 

- More workarounds appear to get around Microsoft's local account restrictions (notebookcheck.net) 


[1]:https://borncity.com/win/2025/10/09/windows-11-setup-microsoft-will-block-the-creation-of-local-accounts-in-the-future/
[2]:https://www.notebookcheck.net/More-workarounds-appear-to-get-around-Microsoft-s-local-account-restrictions.1134227.0.html