---
title: 'Mimick Manjaro Mate using sdboot and luks'
taxonomy:
    category:
        - docs
---

## Toolbox script
Installing a Manjaro Mate using systemd boot and LUKS encryption

```

#
#    This program is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License
#    along with this program.  If not, see <https://www.gnu.org/licenses/>.
#
# Copyright (c) 2021 @linux-aarhus
#
# This script is based on
#    https://forum.manjaro.org/t/root-tip-diy-installer-script-base-sdboot-luks/88502
#
# MODIFIED TO FIT A SPECIFIC FORUM TOPIC
#    https://forum.manjaro.org/t/how-do-i-install-manjaro-mate-with-lvm-on-luks-and-dual-boot-with-windows/87358
#
#  ! IMPORTANT !
#  ! GREAT CARE SHOULD BE EXCERCISED !
#  ! A lot of assumptions are made - please review carefully !
#
#    The script assumes the system is booted using a recent Manjaro Mate ISO
#    The disk is pre-installed using Windows in EFI mode
#    The Manjaro ISO is booted in EFI mode (firmware Legacy boot disabled)
#    The existing Windows EFI partition ($esp) is located at /dev/sda1
#    An empty partition has been created to hold a Linux filesystem (0x8300) as /dev/sda5
#    systemd-boot is installed as bootloader
#    The root filesystem is using f2fs inside a LUKS container
#
#  ! PLEASE REVIEW THE VARIABLES SECTION !
#  ! AMEND THE VARIABLES AS NECESSARY !
#

if [ "$(id -u)" != "0" ]; then
    echo "Please change to root context using su or sudo"
    echo ""
    exit
fi

#############################################################
#### VARIABLES SECTION

TARGET="/dev/sda"
EFI_PART="/dev/sda1"                         # existing Windows $esp
LUKS_PART="/dev/sda2"                        # root partition to hold LUKS container
TUSER=manjaro                                # first user == wheel group
TDISPLAYMANAGER=lightdm                      # display manager
KERNEL="5.10"                                # linux kernel number
KERNELPKG=$(echo linux$KERNEL | sed 's/\.//')# kernel package name
MIRROR='https://mirrors.manjaro.org/repo/'   # build mirror
BRANCH='unstable'                            # target branch
TKEYMAP='dk'                                 # target keyboard layout
TLOCALE_CONF='en_DK.UTF-8'                   # target locale.conf
TLOCALE_PRIMARY='en_DK.UTF-8 UTF-8'          # target primary locale
TLOCALE_FALLBACK='en_US.UTF-8 UTF-8'         # target fallback locale
TTIMEZONE='Europe/Copenhagen'                # target timezone
THOSTNAME='manjaro'                          # target hostname
ITER_TIME="10000"                            # luks iteration time
RETRIES="3"                                  # luks decryption retries
BASE_PKGS="base $KERNELPKG mkinitcpio networkmanager bash-completion"
TGROUPS='lp,network,power,wheel'
TSERVICES='cronie ModemManager NetworkManager cups tlp tlp-sleep avahi-daemon add-autologin-group haveged apparmor snapd.apparmor snapd'

#### VARIABLES END
#############################################################

# == BARE METAL TEST SETUP ============================
#echo "==> Unmounting $TARGET"
#umount -f "$TARGET"
#echo "==> Preparing disk $TARGET"
#sgdisk --zap-all "$TARGET"
#sgdisk --mbrtogpt "$TARGET"
#### efi
#echo "==> Creating EFI partition"
#sgdisk --new 1::+512M  --typecode 1:ef00 --change-name 1:"EFI System" "$TARGET"
#echo "==> wiping EFI partition"
#wipefs -af "$TARGET"1
#echo "==> formatting EFI partition"
#mkfs.vfat -F32 "$TARGET"1
#### root
#echo "==> Creating root partition"
#sgdisk --new 2::: --typecode 2:8304 --change-name 2:"Linux x86-64 root" "$TARGET"
#echo "==> wiping root partition"
#wipefs -af "$TARGET"2
# == END BARE METAL TEST SETUP ========================

# ===== EXISTING WINDOWS DEVICE =======================
echo "==> Unmounting $EFI_PART"
umount -f "$EFI_PART"
echo "==> Unmounting $LUKS_PART"
umount -f "$LUKS_PART"
echo "==> wiping root partition"
wipefs -af "$LUKS_PART"
# ====================================================

echo "==> ------------------------------------------"
echo "==> Setting up root LUKS container"
echo "  -> WATCHOUT FOR THE UPPERCASE CONFIRMATION"
echo "  -> If using CapsLock remember to toggle back"
cryptsetup --type luks2 --use-urandom luksFormat "$LUKS_PART"

echo "==> ------------------------------------------"
echo "==> Open LUKS container"
cryptsetup open "$LUKS_PART" cryptroot

echo "==> Formatting LUKS using ext4"
mkfs.ext4 /dev/mapper/cryptroot

echo "==> Mounting root partition"
mount /dev/mapper/cryptroot /mnt

echo "==> Creating /boot"
mkdir /mnt/boot

echo "==> Mounting EFI partition"
mount "$EFI_PART" /mnt/boot

echo "==> Setting branch and mirror"
pacman-mirrors --api --set-branch $BRANCH --url $MIRROR

echo "==> Syncronizing pacman databases"
pacman -Syy

echo "==> installing base system"
basestrap /mnt $BASE_PKGS

echo "==> Configure base ..."
echo "  -> Creating file: vconsole.conf"
echo KEYMAP=$TKEYMAP > /mnt/etc/vconsole.conf

echo "  -> Creating file: locale.conf"
echo LANG=$TLOCALE_CONF > /mnt/etc/locale.conf

echo "  -> Creating file: hostname"
echo manjaro > /mnt/etc/hostname

echo "  -> Creating file: hosts"
cat > /mnt/etc/hosts <<EOF
127.0.0.1 localhost
127.0.1.1 $THOSTNAME.localdomain $THOSTNAME
EOF

echo "  -> Creating symlink: localtime"
ln -sf /usr/share/zoneinfo/$TTIMEZONE /mnt/etc/localtime

echo "  -> Setting hardware clock"
manjaro-chroot /mnt hwclock --systohc

echo "  -> Enabling services"
manjaro-chroot /mnt  systemctl enable NetworkManager systemd-timesyncd

echo "  -> Modifying file: locale.gen"
echo $TLOCALE_PRIMARY >> /mnt/etc/locale.gen
echo $TLOCALE_FALLBACK >> /mnt/etc/locale.gen

echo "  -> Generating locale"
manjaro-chroot /mnt locale-gen

echo "  -> Setting up mkinitcpio.conf"
sed -i '/HOOKS=/c\HOOKS=(systemd keyboard keymap sd-vconsole block sd-encrypt autodetect modconf filesystems fsck)' /mnt/etc/mkinitcpio.conf

echo "  -> Generating initrd"
manjaro-chroot /mnt mkinitcpio -P

echo "  -> Installing bootloader"
bootctl --path=/mnt/boot install

echo "  -> Updating entries with device UUID"
devuuid=$(lsblk -no uuid "$LUKS_PART" | head -n1)

echo "  -> Creating loader entry: manjaro.conf"
cat > /mnt/boot/loader/entries/manjaro.conf <<EOF
title   Manjaro
linux   /vmlinuz-$KERNEL-x86_64
initrd  /initramfs-$KERNEL-x86_64.img
options root=/dev/mapper/cryptroot rd.luks.name=$devuuid=cryptroot
EOF

echo "  -> Creating fallback entry: manjaro-fallback.conf"
cat > /mnt/boot/loader/entries/manjaro-fallback.conf <<EOF
title   Manjaro (fallback)
linux   /vmlinuz-$KERNEL-x86_64
initrd  /initramfs-$KERNEL-fallback-x86_64.img
options root=/dev/mapper/cryptroot rd.luks.name=$devuuid=cryptroot
EOF

echo "  -> Setting default loader"
sed -i '/default/c\/default manjaro\*/' /mnt/boot/loader/loader.conf

echo "==> Setting target branch and mirror"
pacman-mirrors --api --prefix /mnt --set-branch $BRANCH --url $MIRROR

echo "==> Set root password"
manjaro-chroot /mnt passwd root

#############################################################
#### ISO SPECIFIC SETUP

echo "==> Installing display manager"
manjaro-chroot /mnt pacman -S $TDISPLAYMANAGER --noconfirm

echo "==> Installing ISO package lists"
manjaro-chroot /mnt pacman -S $(comm -12 <(awk '{print $1}' /rootfs-pkgs.txt | sort) <(awk '{print $1}' /desktopfs-pkgs.txt | sort) | sed '/^grub/d' | sed '/^os-prober/d' | sed '/^kernel-modules-hook/d' | sed '/^kernel-alive/d' | sed '/^linux[0-9][0-9]/d') --needed --noconfirm
echo " --> Done installing ISO packages."

echo "==> Copying ISO specific settings..."
cp /etc/lightdm/lightdm-gtk-greeter.conf /mnt/etc/lightdm
cp /etc/lightdm/slick-greeter.conf /mnt/etc/lightdm
cp /etc/environment /mnt/etc
cp /usr/share/icons/default /mnt/usr/share/icons

echo "==> Setting up wheel group"
cat > /mnt/etc/sudoers.d/100-wheel <<EOF
%wheel ALL=(ALL) ALL
EOF

echo "==> Create new admin user $TUSER"
manjaro-chroot /mnt useradd -mUG $TGROUPS $TUSER

echo "==> Set admin user password"
manjaro-chroot /mnt passwd $TUSER

echo "==> Enable display manager"
manjaro-chroot /mnt systemctl enable $TDISPLAYMANAGER

echo "==> Enable ISO services"
manjaro-chroot /mnt systemctl enable $TSERVICES

#### ISO SETUP END
#############################################################

echo "==> Cleaning up"
echo "  -> Unmounting partitions"
umount -R /mnt

echo "  -> Closing LUKS container"
cryptsetup close /dev/mapper/cryptroot
sync

echo "==> Done! You have succesfully mimicked a Manjaro Mate Edition"
echo "==> TODO: Configure swapfile ..."
echo "  -> Swap configuration <https://wiki.manjaro.org/index.php/Swap>"
echo ""

```