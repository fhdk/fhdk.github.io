---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Thunar custom actions'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

# Add Thunar custom actions
---
As a maintainer I have made some custom actions for Thunar.

## What we will cover
---
1. Write ISO to USB
2. Open a terminal using the current folder
3. Launching Thunar as administrator
4. Edit configuration as admin using GUI

## How?
---
All custom actions are defined using the **Configure custom actions...** dialog in Thunar. It is found in the **Edit** menu.

All examples will follow the pattern

* Navigate the **Edit menu**
* Click **Configure custom actions...**
* Click **+** to add an action
* Configure the action
* Save the action

Only the first example will detail the action of opening the *Configure custom action* dialog

## Write ISO to USB
---
For ISO writing I have a preference for Mint stick. I am thinking it is because it does 2 two things and does it well. There is several benefits using Mint stick

* Mint stick only allows USB devices
* USB device formatting
* USB image writing

Install `mintstick` package

    sudo pacman -Syu mintstick

Open Thunar and navigate to **Edit** &rarr; **Configure custom actions...**

* Click the **+** add an action
  * Name: **Write to USB**
  * Description: **Write selected ISO to USB**
  * Command: **mintstick -m iso -i %f**

* Select a nice icon by clicking the icon e.g. 
  - from the **Device icons** dropdown locate the **media-flash-memory-stick** icon.

* Click the **Appearance Conditions** tab
  * File patter: ***.iso**
  * Select only the **Other Files** option
  * Shortcut: Click and press e.g. <kbd>F7</kbd>
Click **OK** and close the window.

Test it by navigating to a folder containing an ISO. Then right click the ISO and select the new **Write to USB** option.

Close Mint stick and test selected shortcut e.g. <kbd>F7</kbd>.

## Open terminal here
---
Opening a terminal in a folder is probably one of the most useful features of the file manager. You can use a named terminal or you can use the `exo-open` app to launch the system's terminal defined by **Preferred applications**. To be able to use this you need the package `exo` installed. You can check if it is installed using `which exo-open`.

Click **+** in the custom actions dialog to configure a new action
* Name: **Open Terminal Here**
* Description: **Open terminal in current folder**

How do we get it to open in the folder currently open in Thunar? 
At the bottom of the **Basics** tab is a list of parameters you can use when executing a command.

In this case we want the current folder to be the location opened in the terminal.

We look at the list and find the **%d** parameter
> ```
>%d    directory containing the file that is passed in %f
>```
Another parameter is referenced **%f** and this is exactly what we need 
> ```
>%f    the path to the first selected file or directory
>```

So our command will be

* Command: **exo-open --launch TerminalEmulator %d**

Next is shortcut
* Keyboard Shortcut: <kbd>F4</kbd>

And icon
* Select a suitable icon - a terminal

Now move to the **Appearance Conditions** tab
* File Pattern: **`*`**
* Select only the **Directories** option

Save the action by clicking **OK**

## Launch Thunar as root
---
Click **+** in the custom actions dialog to configure a new action
* Name: **Open Thunar as ROOT**
* Description: **Open Thunar as ROOT**
* Command: **thunar admin:%d**

The **admin:** is a protocol which allows for tasks to be launched with administrative privileges. This is part of the **gvfs** package (**G**nome **V**irtual **F**ile **S**ystem). We are again using the **%d** parameter to use the current folder for root actions. 

* Keyboard Shortcut: <kbd>Ctrl</kbd><kbd>F8</kbd>

And icon
* Select a suitable icon - red folder as a consequence alert

Now move to the **Appearance Conditions** tab
* File Pattern: **`*`**
* Select **Directories**, **Text Files** and **Other Files** option

Save the action by clicking **OK**

## Edit config as root using GUI
---
Click **+** in the custom actions dialog to configure a new action
* Name: **Edit file as ROOT**
* Description: **Edit file as ROOT**
* Command: **xed admin:%f**

The command should be an editor supporting the admin protocol. This is true for `xed` and `gedit`.  We use the **%f** parameter to pass the full path of the selected file to the editor

* Keyboard Shortcut: <kbd>None</kbd>

And icon
* Select a suitable icon - something signifying the consequence of the action

Now move to the **Appearance Conditions** tab
* File Pattern: **`*`**
* Select only the **Text Files** option

Save the action by clicking **OK**

### What if ...
The editor I want to use does not support the admin protocol?

You can use pkexec and add some environment variables - e.g. if you want to launch geany the command could look like this

    pkexec env DISPLAY=$DISPLAY XAUTHORITY=$XAUTHORITY geany %f

This is a work around - and may change in the future - who knows ...


## Recreate on other systems
---
Creating the actions from scratch is a tedious task - if you have many. The easy way is to copy this file to another system.

> ~/.config/Thunar/uca.xml

## Conclusion
---
You have now learned how you create custom actions in Thunar. I hope you find the guide useful.

You can find inspiration for custom actions in this thread

* https://archived.forum.manjaro.org/t/thunar-custom-actions-are-awesome-what-are-yours/100835?u=linux-aarhus
