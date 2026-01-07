---
title: 'Notes on systemd-boot'
taxonomy:
    category:
        - docs
---

```
mount /dev/sdxY /efi
```

```
bootctl install --esp-path=/efi
```

```
KERNEL-VERSION=$(uname -r)
```

```
kernel-install add $KERNEL-VERSION /usr/lib/modules/$KERNEL-VERSION/vmlinuz
```

```
kernel-install remove $KERNEL-VERSION
```

