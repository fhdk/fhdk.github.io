---
title: 'Boot ISO from GRUB'
taxonomy:
    category:
        - docs
---

```
menuentry "Manjaro  grub_iso"  {
    set isofile="/miso/manjaro-gnome-20.1-stable-x86_64.iso"
    set dri="free"
    set lang="en_US"
    set keytable="fi"
    set timezone="Europe/Helsinki"
    search --no-floppy -f --set=root $isofile
    probe -u $root --set=abc
    set pqr="/dev/disk/by-uuid/$abc"
    loopback loop $isofile
    linux  (loop)/boot/vmlinuz-x86_64  img_dev=$pqr img_loop=$isofile driver=$dri tz=$timezone lang=$lang keytable=$keytable copytoram
    initrd  (loop)/boot/intel_ucode.img (loop)/boot/initramfs-x86_64.img
}
```

Source: [Manjaro Forum](https://forum.manjaro.org/t/howto-booting-manjaro-iso-directly-with-grub/15892)