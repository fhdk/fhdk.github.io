---
title: 'REISUB / REISUO'
taxonomy:
    category:
        - docs
---

Original post by [@đabby][1] Manjaro Forum

**Difficulty: ★★☆☆☆**

REISUB (also known as the [magic SysRq key][2]) is a mnemonic for:

* **Rise up**
* **R**eboot **E**ven **I**f **S**ystem **U**tterly **B**roken
* **R**aising **E**lephants **I**s **S**o **U**tterly **B**oring

and is a *gentle* way of rebooting your system by doing

1. Switch the keyboard from **R**aw mode, used by programs such as [X11 ][3] and [SVGALib][4], to XLATE (translate) mode.
2. Send an **E**nd signal (SIGTERM) to all processes, except the boot process, allowing all processes to end gracefully.
3. Send an **I**nstant kill (SIGKILL) to all processes, except the boot process,  *forcing* all processes to end.
4. **S**ync all mounted filesystems, allowing them to write all data to disk.
5. **U**nmount and remount all mounted filesystems in [read-only][5] mode.
6. Re**B**oot the system

In fact, the only data you lose is since your last [auto]save as the system is shut down gracefully.

Before you read any further, ensure all your work is saved!

**How to invoke the REISUB procedure?**
First we have to unlock the SysRq key so:

* Add to your grub `GRUB_CMDLINE_LINUX_DEFAULT` the `sysrq_always_enabled=1` variable.
* Execute `echo kernel.sysrq=1 | sudo tee --append /etc/sysctl.d/99-sysctl.conf`
* Reboot normally if you had to add one of these parameters and come back here to read the rest of the blurb...

There is a special **Sys**tem **R* e**q**uest key on most keyboards called <kbd>SysRq</kbd> on all English language and most international keyboards.

<kbd>Stamp</kbd> (or Stampa on old Olivetti keyboards)
<kbd>S-Abf</kbd>

* on full-size keyboards it's called by pressing <kbd>Alt</kbd>+<kbd>SysRq</kbd>
* on most laptop keyboards it's called by pressing <kbd>Fn</kbd>+<kbd>Alt</kbd>+<kbd>PrtSc</kbd>
* On some keyboards, please also try <kbd>Alt</kbd>+<kbd>PrtSc</kbd> if the <kbd>SysRq</kbd> key is missing or <kbd>Alt Gr</kbd>+<kbd>SysRq</kbd> if none of the above <kbd>Alt</kbd> combinations work .

Once you've located your <kbd>SysRq</kbd> key, please *keep the* <kbd>Alt</kbd> *key pressed*.

### Test 
Now lightly tap these keys *waiting between 1 second (fast, new machines) and 6 seconds (older or resource-starved machines¹) in-between keypresses* : <kbd>R</kbd><kbd>E</kbd><kbd>I</kbd><kbd>S</kbd><kbd>U</kbd><kbd>B</kbd>

If you actually followed the above instructions, your system just rebooted gracefully and you did not damage your HDD (if you still have one) by forcing it into PARK mode forcefully nor was your EXT4 or NTFS transaction journal rolled back.

* For even more info, [Read the Fine Manual ][6]
* If a REISUB just restarts your DE just do a REISUO after the REISUB.

1. Read the REISUB section above and then you'll know what this means:
2. <kbd>R</kbd><kbd>E</kbd><kbd>I</kbd><kbd>S</kbd><kbd>U</kbd><kbd>O</kbd> will turn your machine off instead of rebooting it...
 
1. Sometimes (If only the UI is frozen), you can still switch to one of the TTYs by pressing <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F2</kbd> or <kbd>F3</kbd>, ... and gently tap the power button there.
2. To do the same from within your Desktop Environment (DE) *this has to be set up beforehand!*
So to ensure that your power button effectively shuts down your computer instead of going to sleep / Hibernate / ... *in the future* , follow these steps for your DE:
* In Gnome: 
   * Execute the following command:
   ```
     gsettings set org.gnome.settings-daemon.plugins.power button-power 'shutdown'
   ```
* In KDE: 
   * Go to <kbd>System Settings</kbd>
   * Type `energy`
   * Click <kbd>Energy Saving</kbd>
   * in all of the <kbd>On AC Power</kbd>, <kbd>On Battery</kbd> and <kbd>On Battery</kbd> tabs, set the check-mark on <kbd>Button events handling</kbd> and the <kbd>Even when an external monitor is connected</kbd>
   * <kbd>When power button pressed</kbd> to `Shut Down`
   * Click <kbd>Apply</kbd>
* In XFCE:
     * Go to the Applications menu or <kbd>System settings</kbd>
     * Clock <kbd>Power Manager</kbd>
     * From <kbd>When Power Button is pressed:</kbd> select <kbd>Shutdown</kbd>.

[1]: https://forum.manjaro.org/t/howto-reboot-turn-off-your-frozen-computer-reisub-reisuo/3855
[2]: https://wiki.archlinux.org/index.php/Keyboard_shortcuts#Kernel
[3]: https://en.wikipedia.org/wiki/X11
[4]: https://en.wikipedia.org/wiki/SVGALib
[5]: https://en.wikipedia.org/wiki/File_system_permissions
[5]: https://wiki.archlinux.org/index.php/Keyboard_shortcuts#Kernel