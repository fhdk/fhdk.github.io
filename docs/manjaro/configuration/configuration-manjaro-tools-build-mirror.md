---
title: 'Manjaro Tools build_mirror'
date: '11:33 20-12-2022'
taxonomy:
    category:
        - docs
---

To make the tools config follow primary mirror in mirrorlist

Edit your **~/.config/manjaro-tools/manjaro-tools.conf**

```
build_mirror=$(grep /etc/pacman.d/mirrorlist -e 'http' | head -n 1 | cut -d' ' -f3- | rev | cut -d'/' -f4- | rev)
```