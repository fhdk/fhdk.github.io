---
title: 'Locale error messages'
taxonomy:
    category:
        - docs
published: true
date: '2020-09-04 08:01'
publish_date: '2020-09-04 08:01'
---

## Locale error message

When you get errors like these examples - it is not obvious what causes it

```
$ rofi

(process:1628): Rofi-WARNING **: 07:34:43.942: Failed to set locale.
```
```
$ sterminal
couldn't read from shell: Input/output error
child exited with status 1
tmux: invalid LC_ALL, LC_CTYPE or LANG
```

## Check your settings
!!! Obviously you need to replace the locale with something relevant for your system. 
!!! I have written this using my system's locale - **en_DK** for messages and **da_DK** for the rest of the system.

Check locale settings
```
$ localectl      
   System Locale: LANG=en_DK.UTF-8
                  LC_NUMERIC=da_DK.UTF-8
                  LC_TIME=da_DK.UTF-8
                  LC_MONETARY=da_DK.UTF-8
                  LC_PAPER=da_DK.UTF-8
                  LC_NAME=da_DK.UTF-8
                  LC_ADDRESS=da_DK.UTF-8
                  LC_TELEPHONE=da_DK.UTF-8
                  LC_MEASUREMENT=da_DK.UTF-8
                  LC_IDENTIFICATION=da_DK.UTF-8
       VC Keymap: dk-latin1
      X11 Layout: dk
       X11 Model: pc105
```

Check installed locales

```
$ locale -a
C
en_DK.utf8
en_US.utf8
POSIX
```

The installed locales does not match the settings listed by `localectl` - as you can see the **da_DK** part is missing.

## Fix locale error

It is recommended to use the utf8 version unless you have compelling reasons to select otherwise.

### Method 1
Edit `/etc/locale.gen` and ensure that all in-use locales has been uncommented. For fallback messages enable **en_US** as well

```bash
$ sudo nano /etc/locale.gen
```

```text
...
#cy_GB ISO-8859-14  
da_DK.UTF-8 UTF-8  
#da_DK ISO-8859-1  
....
#en_CA ISO-8859-1  
en_DK.UTF-8 UTF-8  
#en_DK ISO-8859-1  
...
#en_SG ISO-8859-1  
en_US.UTF-8 UTF-8  
#en_US ISO-8859-1  
...
```

### Method 2
Based on the comment by @nam1962 (see [[2]]).

Check if the locale you want to use is available in the locale list (**/etc/locale.gen**)

```
$ cat /etc/locale.gen | grep 'da_DK'
#da_DK.UTF-8 UTF-8  
#da_DK ISO-8859-1
```

Use `sed ` command

* To enable a locale (uncommenting the line)
   ```
   $ sudo sed -i '/en_DK.UTF-8/s/^#//g' /etc/locale.gen
   $ sudo sed -i '/en_US.UTF-8/s/^#//g' /etc/locale.gen
   ```

* To disable a locale (aka commenting the locale)
   ```
   $ sudo sed -i '/en_DK.UTF-8/s/^/#/g' /etc/locale.gen
   ```

### Rebuild locales

```bash
$ sudo locale-gen
Generating locales...
  da_DK.UTF-8... done
  en_DK.UTF-8... done
  en_US.UTF-8... done
Generation complete.
```

### Recheck your locales

```
$ locale -a      
C
da_DK.utf8
en_DK.utf8
en_US.utf8
POSIX
```

### Verify it works

```
➜  ~ rofi -r
Rofi is unsure what to show.
Please specify the mode you want to show.

    rofi -show {mode}

The following modi are enabled:
 * window
 * run
 * ssh

The following can be enabled:
 * windowcd
 * drun
 * combi
 * keys

To activate a mode, add it to the list of modi in the modi setting.
```

## Now it is fixed
Enjoy

[2]: https://forum.manjaro.org/t/howto-troubleshooting-locale-errors/21008/3?u=linux-aarhus