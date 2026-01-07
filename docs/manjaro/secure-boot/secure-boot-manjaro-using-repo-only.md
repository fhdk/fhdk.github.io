---
title: 'Manjaro and Windows - Secure Boot - repo only'
date: '15:42 21-02-2025'
taxonomy:
    category:
        - docs
---

## About Secure Boot

- Secure Boot is a mechanism for verifying operating system integrity
- The mechanism was pionereed by Microsoft and is now a defacto standard
- Configuring Secure Boot requires access to the system's firmware
- The firmware access requirement prevent OOB support of Secure Boot

## Background

This How-To is based on the question asked with [Enable Secure Boot For Existing Manjaro Usung Repo Only] and can be seen as a complement to [Encrypted Manjaro Linux Using Verified Boot].

I decided to work on the theory - and brush up on new sbctl options and configuration.

To be able to comply with corporate security - this is a proof-of-concept - how to make Manjaro and Windows co-exist with Secure Boot enabled.

## Precaution

Make sure your system’s firmware is up-to-date - if in doubt check with the vendor’s website.

Your system may require a signed driver blob stored in the firmware - in such case contact your vendor through their official support channels - and verify if you void your warranty or brick your system.

**You are the system admin**

> THE PROOF OF CONCEPT IS PROVIDED “AS IS” AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS DOCUMENT INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OF THIS PROOF OF CONCEPT.

## Target System

The test system is Clevo N141wu with a single NVMe storage device and the steps I am going to list works with the mentioned laptop.

I downloaded and installed a Windows 10 - and subsequently added Manjaro using GRUB for dual-boot using a single Linux 6.12 kernel.

Although this project did not test Windows 11, there is no reason to believe the process would be any different than the one applied herein.

If you are trying to implement this with another layout - you must adapt the process.

Other systems tested (removed existing key database)

- Thinkpad X13 AMD Gen.4
- Tuxedo InfinitiBookPro 14 Gen.8

## Proof of Concept

This is a practical implementation using **sbctl** on Manjaro Linux.

See -> https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Assisted_process_with_sbctl

The PoC assumes the system has been kept up-to-date both operating systems.

Sync the package sbctl from the official repo

```
sudo pacman -S sbctl
```

After this you have an excellent opportunity to familiarize yourself with **sbctl** usage

```text
sbctl --help
sbctl bundle --help
sbctl create-keys --help
sbctl enroll-keys --help
sbctl generate-bundles --help
```

### Kernel Cmdline

Extract the *GRUB_CMDLINE_LINUX_DEFAULT* from `/etc/default/grub` and save the content in a **new file `/etc/kernel/cmdline`**. The content - using a default Manjaro installation - would look like

```
root=UUID=<uuid> rw quiet splash apparmor=1 security=apparmor udev.log_priority=3
```

When you created the file - create a new bundle using **sbctl**. 

### Define Bundle

A bundle is the set of files used to start your system.

To create a bundle definition you run **`sbctl bundle <args>`** providing the mountpoint of the $esp, paths to the files to bundle and full path to the unified efi loader.

- `--amducode '/boot/amd-ucode.img'` includes ucode for AMD
- `--intelucode  '/boot/intel-ucode.img'`includes ucode for Intel

Example for an AMD system

```
sudo sbctl bundle --esp '/boot/efi' \
                  --amducode '/boot/amd-ucode.img' \
                  --cmdline '/etc/kernel/cmdline' \
                  --initramfs '/boot/initramfs-6.12-x86_64.img' \
                  --kernel-img '/boot/vmlinuz-6.12-x86_64' \
                  --save '/boot/efi/main.efi'
```

As can be deduced - it is fairly easy to create extra bundles with e.g. a fallback image - **just don't do it** (See Storage Consideration below).

### Create Signing Keys

A key is required to sign the bundle - so create the key

```
sudo sbctl create-keys
```

### Generate Bundle and Sign

When you have created the keys - generate the bundle and sign

```
sudo sbctl generate-bundles --sign
```

### Configure Setup Mode

Restart your system and enter the firmware setup

```
systemctl reboot --firmware-setup
```

In the system firmware you need to locate the **Secure Boot** section, then configure Secure Boot for **Setup Mode**. 

How you do this is specific for your firmware - you may need to experiment.

### Enroll Keys

When you are confident the system's Secure Boot is in Setup Mode, boot into your Manjaro system - and enroll your key into Secure Boot key storage, remember to include Microsoft keys

```
sudo sbctl enroll-keys --microsoft
```

The command ensures the firmware Setup Mode is reverted to production mode. It is not automagically protected - for that you need to set an administrative password.

## Firmware boot entry

To be able to use your firmware's boot override it is necesary to create a firmware boot entry. For our PoC the command look like this

To avoid confusing the default Manjaro entry; label the entry **Manjaro Verified Boot**

```
sudo efibootmgr  --loader main.efi --create --disk /dev/nvme0n1 --part 1 --label 'Manjaro Verified Boot' --unicode
```

## Storage Consideration

The $esp (efi system partition) with default Microsoft Windows 10 installation is 100M, and this is leaving little room to wiggle, as a single efi image takes 42M, thus leaving 29M remaining space.

```text
[nix-n14xwu ~]# ls -l -h /boot/efi
total 42M
drwx------ 5 root root 1,0K 21 feb 12:59  EFI
drwx------ 2 root root 1,0K 21 feb 13:35  loader
-rwx------ 1 root root  42M 21 feb 12:42  main.efi
drwx------ 2 root root 1,0K 21 feb 11:37 'System Volume Information'
```

## Maintenance

The bundle is a static configuration - it will not change automagically - e.g. triggered by a hook.

The kernel maintenance within a release e.g. 6.12 is not an issue as the name has not changed.

When you decide to switch kernel e.g. from 6.12 to 6.14 - the name changes and you need to maintain your bundle.

### Generate new Bundle
This can be done by recreating your bundle configuration using **sbctl**

```
sudo sbctl bundle --esp '/boot/efi' \
                  --amducode '/boot/amd-ucode.img' \
                  --cmdline '/etc/kernel/cmdline' \
                  --initramfs '/boot/initramfs-6.14-x86_64.img' \
                  --kernel-img '/boot/vmlinuz-6.14-x86_64' \
                  --save '/boot/efi/main.efi'
```

### Manually update bundle

The bundle configuration is stored in `/var/lib/sbctl/bundles.json` so you can manually change the filenames but then you have no error checking if you misspell a filename - thus it is not recommended - instead use the bundle command.

### Generate bundle 

Generate the bundle and sign it

```
sudo sbctl generate-bundles --sign
```

## Protect Your Firmware

It is strongly suggested to lock your computers firmware with an administrative password - otherwise it is easy to circumvent the boot measures applied.

The firmware will now be enabled and your system may now be compliant with your corporate policies stating Secure Boot must be enabled.

Before you rejoice too much - ask your Corporate IT department.
## About Secure Boot

- Secure Boot is a mechanism for verifying operating system integrity
- The mechanism was pionereed by Microsoft and is now a defacto standard
- Configuring Secure Boot requires access to the system's firmware
- The firmware access requirement prevent OOB support of Secure Boot

## Background

This How-To is based on the question asked with [Enable Secure Boot For Existing Manjaro Usung Repo Only] and can be seen as a complement to [Encrypted Manjaro Linux Using Verified Boot].

I decided to work on the theory - and brush up on new sbctl options and configuration.

To be able to comply with corporate security - this is a proof-of-concept - how to make Manjaro and Windows co-exist with Secure Boot enabled.

## Target System

The test system is Clevo N141wu with a single NVMe storage device and the steps I am going to list works with the mentioned laptop.

I downloaded and installed a Windows 10 - and subsequently added Manjaro using GRUB for dual-boot using a single Linux 6.12 kernel.

Although this project did not test Windows 11, there is no reason to believe the process would be any different than the one applied herein.

If you are trying to implement this with another layout - you must adapt the process.

## Proof of Concept

This is a practical implementation using **sbctl** on Manjaro Linux.

See -> https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Assisted_process_with_sbctl

The PoC assumes the system has been kept up-to-date both operating systems.

Sync the package sbctl from the official repo

```
sudo pacman -S sbctl
```

After this you have an excellent opportunity to familiarize yourself with **sbctl** usage

```text
sbctl --help
sbctl bundle --help
sbctl create-keys --help
sbctl enroll-keys --help
sbctl generate-bundles --help
```

### Kernel Cmdline

Extract the *GRUB_CMDLINE_LINUX_DEFAULT* from `/etc/default/grub` and save the content in a **new file `/etc/kernel/cmdline`**. The content - using a default Manjaro installation - would look like

```
root=UUID=<uuid> rw quiet splash apparmor=1 security=apparmor udev.log_priority=3
```

When you created the file - create a new bundle using **sbctl**. 

### Define Bundle

A bundle is the set of files used to start your system.

To create a bundle definition you run **`sbctl bundle <args>`** providing the mountpoint of the $esp, paths to the files to bundle and full path to the unified efi loader.

- `--amducode '/boot/amd-ucode.img'` includes ucode for AMD
- `--intelucode  '/boot/intel-ucode.img'`includes ucode for Intel

Example for an AMD system

```
sudo sbctl bundle --esp '/boot/efi' \
                  --amducode '/boot/amd-ucode.img' \
                  --cmdline '/etc/kernel/cmdline' \
                  --initramfs '/boot/initramfs-6.12-x86_64.img' \
                  --kernel-img '/boot/vmlinuz-6.12-x86_64' \
                  --save '/boot/efi/main.efi'
```

As can be deduced - it is fairly easy to create extra bundles with e.g. a fallback image - **just don't do it** (See Storage Consideration below).

### Create Signing Keys

A key is required to sign the bundle - so create the key

```
sudo sbctl create-keys
```

### Generate Bundle and Sign

When you have created the keys - generate the bundle and sign

```
sudo sbctl generate-bundles --sign
```

### Configure Setup Mode

Restart your system and enter the firmware setup

```
systemctl reboot --firmware-setup
```

In the system firmware you need to locate the **Secure Boot** section, then configure Secure Boot for **Setup Mode**. 

How you do this is specific for your firmware - you may need to experiment.

### Enroll Keys

When you are confident the system's Secure Boot is in Setup Mode, boot into your Manjaro system - and enroll your key into Secure Boot key storage, remember to include Microsoft keys

```
sudo sbctl enroll-keys --microsoft
```

The command ensures the firmware Setup Mode is reverted to production mode. It is not automagically protected - for that you need to set an administrative password.

## Firmware boot entry

To be able to use your firmware's boot override it is necesary to create a firmware boot entry. For our PoC the command look like this

To avoid confusing the default Manjaro entry; label the entry **Manjaro Verified Boot**

```
sudo efibootmgr  --loader main.efi --create --disk /dev/nvme0n1 --part 1 --label 'Manjaro Verified Boot' --unicode
```

## Storage Consideration

The $esp (efi system partition) with default Microsoft Windows 10 installation is 100M, and this is leaving little room to wiggle, as a single efi image takes 42M, thus leaving 29M remaining space.

```text
[nix-n14xwu ~]# ls -l -h /boot/efi
total 42M
drwx------ 5 root root 1,0K 21 feb 12:59  EFI
drwx------ 2 root root 1,0K 21 feb 13:35  loader
-rwx------ 1 root root  42M 21 feb 12:42  main.efi
drwx------ 2 root root 1,0K 21 feb 11:37 'System Volume Information'
```

## Maintenance

The bundle is a static configuration - it will not change automagically - e.g. triggered by a hook.

The kernel maintenance within a release e.g. 6.12 is not an issue as the name has not changed.

When you decide to switch kernel e.g. from 6.12 to 6.14 - the name changes and you need to maintain your bundle.

### Generate new Bundle
This can be done by recreating your bundle configuration using **sbctl**

```
sudo sbctl bundle --esp '/boot/efi' \
                  --amducode '/boot/amd-ucode.img' \
                  --cmdline '/etc/kernel/cmdline' \
                  --initramfs '/boot/initramfs-6.14-x86_64.img' \
                  --kernel-img '/boot/vmlinuz-6.14-x86_64' \
                  --save '/boot/efi/main.efi'
```

### Manually update bundle

The bundle configuration is stored in `/var/lib/sbctl/bundles.json` so you can manually change the filenames but then you have no error checking if you misspell a filename - thus it is not recommended - instead use the bundle command.

### Generate bundle 

Generate the bundle and sign it

```
sudo sbctl generate-bundles --sign
```

## Protect Your Firmware

It is strongly suggested to lock your computers firmware with an administrative password - otherwise it is easy to circumvent the boot measures applied.

The firmware will now be enabled and your system may now be compliant with your corporate policies stating Secure Boot must be enabled.

Before you rejoice too much - ask your Corporate IT department.

Crossposted at [Enable Secure Boot For Existing Manjaro Usung Repo Only]

[Enable Secure Boot For Existing Manjaro Usung Repo Only]: https://forum.manjaro.org/t/enable-secure-boot-for-existing-manjaro-using-repo-only/174514
[Manjaro and Windows - Using Secure Boot and Repo Only]: https://forum.manjaro.org/t/root-tip-how-to-manjaro-and-windows-using-secure-boot-and-repo-only/174567
[Encrypted Manjaro Linux Using Verified Boot]: https://forum.manjaro.org/t/root-tip-utility-script-encrypted-manjaro-linux-using-verified-boot/155032
