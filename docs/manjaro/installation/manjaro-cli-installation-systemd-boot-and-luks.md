---
title: 'Manjaro base using sdboot and luks'
taxonomy:
    category:
        - docs
---

## Toolbox script
Installing a base Manjaro using systemd boot and LUKS encryption

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
#  ! IMPORTANT !
#  ! GREAT CARE SHOULD BE EXCERCISED !
#  ! A lot of assumptions are made - please review carefully !
#
#   The root filesystem is using f2fs inside a LUKS container
#
#  ! PLEASE REVIEW THE VARIABLES SECTION !
#  ! AMEND THE VARIABLES AS NECESSARY !
#


if [ "$(id -u)" != "0" ]; then
    echo "Please change to root context using su or sudo"
    echo "Usage: bash sdboot-luks-setup.sh /dev/sdX"
    echo ""
    exit
fi

if [[ -z "$1" ]]; then
    echo "No device specified ..."
    echo "Usage: bash sdboot-luks-setup.sh /dev/sdX"
    echo ""
    exit
fi

TARGET="$1"                               # target device from first argument
KERNEL="5.10"                             # linux kernel number
KERNELPKG=$(echo linux$2 | sed 's/\.//')  # kernel package name
MIRROR='https://repos.nix.dk/manjaro/'    # build mirror
BRANCH='unstable'                         # target branch
TKEYMAP='dk'                              # target keyboard layout
TLOCALE_CONF='en_DK.UTF-8'                # target locale.conf
TLOCALE_PRIMARY='en_DK.UTF-8 UTF-8'       # target primary locale
TLOCALE_FALLBACK='en_US.UTF-8 UTF-8'      # target fallback locale
TTIMEZONE='Europe/Copenhagen'             # target timezone
THOSTNAME='manjaro'                       # target hostname
ITER_TIME="10000"                         # luks iteration time
RETRIES="3"                               # luks decryption retries
BASE_PKGS="base $KERNELPKG mkinitcpio networkmanager f2fs-tools bash-completion"

echo "==> Unmounting $TARGET"
umount -f "$TARGET"

echo "==> Preparing disk $TARGET"
sgdisk --zap-all "$TARGET"
sgdisk --mbrtogpt "$TARGET"

echo "==> Creating EFI partition"
sgdisk --new 1::+512M \
       --typecode 1:ef00 \
       --change-name 1:"EFI System" \
       "$TARGET"

echo "==> Creating root partition"
sgdisk --new 2::: \
       --typecode 2:8304 \
       --change-name 2:"Linux x86-64 root" \
       "$TARGET"

echo "==> wiping EFI partition"
wipefs -af "$TARGET"1

echo "==> wiping root partition"
wipefs -af "$TARGET"2

echo "==> formatting EFI partition"
mkfs.vfat -F32 "$TARGET"1

echo "==> ------------------------------------------"
echo "==> Setting up root LUKS container"
echo "  -> WATCHOUT FOR THE UPPERCASE CONFIRMATION"
echo "  -> If using CapsLock remember to toggle back"
cryptsetup --type luks2 \
           --hash sha512 \
           --iter-time $ITER_TIME \
           --tries $RETRIES \
            --use-urandom luksFormat \
            "$TARGET"2

echo "==> ------------------------------------------"
echo "==> Open LUKS container"
cryptsetup open "$TARGET"2 cryptroot

echo "==> Formatting LUKS using f2fs"
mkfs.f2fs -f /dev/mapper/cryptroot

echo "==> Mounting root partition"
mount /dev/mapper/cryptroot /mnt

echo "==> Creating /boot"
mkdir /mnt/boot

echo "==> Mounting EFI partition"
mount "$TARGET"1 /mnt/boot

echo "==> Setting branch and mirror"
pacman-mirrors --api --set-branch $BRANCH --url $MIRROR

echo "==> Syncronizing pacman databases"
pacman -Syy

echo "==> installing base system with Linux kernel $2"
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
manjaro-chroot /mnt systemctl enable NetworkManager systemd-timesyncd

echo "  -> Modifying file: locale.gen"
echo "$TLOCALE_PRIMARY" >> /mnt/etc/locale.gen
echo "$TLOCALE_FALLBACK" >> /mnt/etc/locale.gen

echo "  -> Generating locale"
manjaro-chroot /mnt locale-gen

echo "  -> Setting up mkinitcpio.conf"
sed -i '/MODULES=/c\MODULES=(f2fs)' \
       /mnt/etc/mkinitcpio.conf
sed -i '/HOOKS=/c\HOOKS=(systemd keyboard keymap sd-vconsole block sd-encrypt autodetect modconf filesystems fsck)' \
       /mnt/etc/mkinitcpio.conf

echo "  -> Generating initrd"
manjaro-chroot /mnt mkinitcpio -P

echo "  -> Installing bootloader"
bootctl --path=/mnt/boot install

echo "  -> Updating entries with device UUID"
devuuid=$(lsblk -no uuid "$TARGET"2 | tail -1)

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

echo "  -> Setup your root password"
manjaro-chroot /mnt passwd

echo "==> Cleaning up"
echo "  -> Unmounting partitions"
umount -R /mnt

echo "  -> Closing LUKS container"
cryptsetup close /dev/mapper/cryptroot
sync

echo "==> Done! Minimal base is now installed and configured"

echo "==> TODO: Install GUI and configure swap"
echo "  -> Manjaro iso profiles <https://gitlab.manjaro.org/profiles-and-settings/iso-profiles>
echo "  -> Swap configuration <https://wiki.manjaro.org/index.php/Swap>"
echo ""

```