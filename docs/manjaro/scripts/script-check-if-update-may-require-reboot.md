---
title: 'Kontroller om genstart er nødvendig før du opdaterer'
date: '06:23 20-11-2022'
taxonomy:
    category:
        - docs
---

**Sværhedsgrad: ★★☆☆☆**
## Beskrivelse
Ved visse system opdateringer **vil** det være nødvendigt at genstarte systemet for at sikre problemfri funktion.

Pakker som indholder *cpu mikrocode*, *kerne*, *drivere*, *xorg* og *systemd* er oplagte emner.

Nedenfor er en liste over pakker som kan kræve en system genstart. Hvis du har andre pakker du ved vil kræve genstart kan du tilføje dem til script variablen **$reboot**

* ucode
* cryptsetup
* linux
* nvidia
* mesa
* systemd
* wayland
* xf86-video
* xorg

## Utility script
Scriptet bruger `checkupdates` og `yay` (for fremmede pakker) hvis den er installeret.

En regex (regular expression) validerer om en system genstart skal foreslåes.

```
#!/usr/bin/env bash
reboot="(ucode|cryptsetup|linux|nvidia|mesa|systemd|wayland|xf86-video|xorg)"
if [[ $(which yay) =~ (yay) ]]; then
    updates=$(checkupdates; yay -Qua)
else
    updates=$(checkupdates)
fi
echo "$updates"
if [[ $updates =~ $reboot ]]; then
    echo "Updating possibly requires system restart ..."
fi
```

## Anvendelse
Scriptet skal ikke betrages som nogen autoritet - det tjener blot det formål at foreslå - før du installerer opdateringer - om en system genstart kan være en nødvendighed.

## Tak til
Inspiration fra @Kresimir og EndeavourOS forum [[1]] [[2]] og det endelige script [[3]]

Publiceret på [Manjaro Form][4]

[1]: https://forum.endeavouros.com/t/check-if-a-reboot-is-neccessary/7092/37
[2]: https://forum.endeavouros.com/t/random-question-how-often-do-you-update-your-packages/7317/13
[3]: https://forum.endeavouros.com/t/random-question-how-often-do-you-update-your-packages/7317/40
[4]: https://forum.manjaro.org/t/root-tip-utility-script-check-if-updates-may-require-system-restart/14112
