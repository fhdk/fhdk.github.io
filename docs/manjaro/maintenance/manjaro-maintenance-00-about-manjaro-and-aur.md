---
title: 'About Manjaro and AUR'
date: '10:02 02-11-2022'
taxonomy:
    category:
        - docs
---

linux-aarhus | 2022-07-24 14:11:34 UTC | Source: [ About Manjaro and AUR](https://forum.manjaro.org/t/need-to-know-about-manjaro-and-aur/103617)

# Manjaro and AUR
This is a superficial review of elements of Linux history spiced with some personal memories and my experience with Arch Linux and Manjaro.

A followup on previous topics

* https://forum.manjaro.org/t/aur-please-restrain-yourself/103318
* https://forum.manjaro.org/t/responsible-use-of-aur/86392

__TL:DR__

<big>custom packages - __unsupported__ - you are on your own!</big>

---

## History part I

The GNU/Linux history lesson is far more than this lesson. This document cannot possibly contain all aspects so only the points I think were necessary to describe the relationship between Manjaro and AUR.

To understand this relationship one need to understand the historical aspects on Linux, source code and distributing binary code.

Applications cannot interact with CPU, RAM and GPU on it own. They all depend on a layer and that layer is the [kernel][1] - in the context of this document the kernel initially created by Linus Torvalds - commonly known as Linux.

The kernel is a collection of services provided in a manner which makes it possible for an application to interaction with the hardware.

If you look at [Linux From Scratch][2] and [Gentoo][3] you will see they are not distributions of binaries but a collection of books describing how to create your operating system from source code.

On the get-started page of the Gentoo web it is expressed as

>The Gentoo Handbook provides detailed documentation that guides you through the installation process.
>
>There is no installation program—you're the installer. That way, you can apply all the customizations you desire.

So - in the beginning there was only source code. You would need to have access to another computer system on which to build the kernel. Before you could do that you would have to, either acquire the source in some form or type it in manually, compile the source to binaries, transfer the binaries to a target system and run it.

If you had a bright idea - e.g. the curl utility - you would share source with the community. They would then acquire the code, compile it to binary form and run it on their system.

A very rough description of the process is something like this

```
curl -O <url/source.tar.gz>
tar -xcf source.tar.gz
cd sourcedir
./configure
make
sudo make install
```
To compile your Linux from source is a tedious, time consuming task - just try it - and this is where distributions are born.

## History part II
### Then what about distributions?
Distributions are groups of individuals or companies which have the power to build a large amount source code into binary form and distribute this as turnkey solutions.

There was Slackware and Red Hat - I specifically remember Red Hat because my younger brother was seriously into Linux before me. SUSE Linux, Slackware and Red Hat were among the first to __distribute binary code__ with SUSE being one of the first commercial distributors of GNU/Linux binary packages. I have very fond memories of SUSE as this is the distribution where I entered the Linux world around 1998.

Four years later (not me) 2002 Arch Linux, then 2011 Manjaro.

Arch got an undeserved reputation of being inaccessible. I dare say - it is not inaccessible.

Imagine a Raspberry PI. It is open source - the schematics are available - you can buy all the components from some well assorted electronics supplier. If you are up to the challenge you buy all the components for the PI and begin putting it together.

At the same electronics supplier you can get a ready-to-run PI or one of the multiple variations.

Using an unjust comparison - Arch Linux the components making up the PI - you just have to put them together - but doing so requires knowledge and hard work. This is where you choose Orange Pi, Banana Pi or something else.

Where am I going you think - well - a single board computer is a single board computer - right?

Yes - and no - even originated in the same idea, being single boarded and working identically, sharing components - they are different.

## What is it with AUR?
To put it short - AUR is 

<big>__Recipes for adding CUSTOM packages__</big>

The name speaks for itself. __A.U.R__ is __Arch__ User Repository - the point being the emphasis on Arch.

Using AUR also implies Arch stable branch - which is only achievable by using Manjaro unstable or testing branch.

As Manjaro is based on Arch Linux one would think - they are the same - and that is where one would go wrong.

Manjaro and Arch __do not__ share kernels, a subtle but very important difference.

Both Manjaro and Arch uses a branch structure (slightly different naming) and builds from the same source at [kernel.org][4] and this is where the similarity ends.

Manjaro maintains a larger number of kernels which uses a slightly different patch-set. You can enlighten yourself on the details on [Manjaro gitlab][5]. Because of this difference certain proprietary drivers which must match the released kernels are different than with Arch.

Some Manjaro applications have been back-ported and is available in AUR - most notably the Pamac Package manager - just search in AUR for pamac.

The AUR maintainer of pamac - is not the developer but a third party - so when the developer pushes new changes - bug fixes, new functionality - anything - the AUR maintainer has to work out if those changes will work on Arch or other Arch based systems.

The same happens the other way around. When one look at the set of build instructions on AUR one need to consider

- the build instructions are for Arch
- the resulting package is for Arch
- what files are the script downloading
- is it source files or binary files
- is the binary files safe
- does the instructions work for Manjaro
- does the build make assumptions only satisfied on Arch (e.g. glibc version)

Being able to find an answer one needs knowledge. Not only on the build process but also knowledge of bash scripting, environment variables, compiling source to binary but knowledge on what actually happens during execution of the build script - which files are replaced, placed where and why.

Because a __CUSTOM package__ build using a script found on AUR has the potential to completely wreck a healthy and well running system AUR has always carried [the warning][6]

> __DISCLAIMER: AUR packages are user produced content. Any use of the provided files is at your own risk.__

## Why the need of responsibility?
Because __CUSTOM packages__  has the potential to wreck your system - they are unsupported. It is not that some scripts are supported and others not.

They are unsupported because they add/alter/replace system files and as such they can generate issues which otherwise would not have occurred on a pristine Manjaro installation.

When you sync your system __any CUSTOM package__ may cease to function without warning __OR__ your system may cease to function due to a __CUSTOM package__ - therefore one must remove all __CUSTOM packages__ before an issue can be validated as related to Manjaro.

If apply system wide changes to your Manjaro system using an unsupported source build script and subsequently the system breaks after an update - you must assume that your changes caused it - you cannot imply it is an upstream problem.

Always bring your own house in order - then you seek help.

## I have AUR packages in my system, but i don't recall having installed them.
Once in a while, package maintainers, both at Arch and Manjaro, clean up the repositories from packages considered "deprecated", "unused", "outdated", etc.

Such packages, when removed from Arch repositories -- and then consequentially from Manjaro repositories -- are usually moved to AUR. Meanwhile, such packages previously available in Manjaro repositories but not in Arch's, are usually available in AUR to begin with.

If you find such packages in your system, you can check if they are either still used (directly), needed (as dependencies), or required (as dependencies of a used package). Depending on the output, you may or may not want to remove them.
```bash
pactree <package>
```


## Applying CUSTOM packages
Like Orange PI is not Raspberry PI - Manjaro is not Arch. You should therefore carefully consider the impact on your system now and future.

- Is this necessary for my workflow
- Remove unused packages
- How is it affecting mys system

The native method to create packages is by means of PKGBUILD script.

This also goes for apps or services not available in the repositories. You write your own PKGBUILD script and learning to so is the best way to understand what AUR is.

To avoid duplicating each others work PKGBUILDs are stored in a central location - the AUR. This storage also make a PKGBUILD easier to keep updated when it is use by many people - therefore the vote system - which to the Arch Community serves as a pointer to a potential inclusion in the official community repository.

Any passing stranger can register an account and create a PKGBUILD which conforms to the [guidelines][7] which is why the following is the initial method to make use of such unverified script from a complete stranger.

Ensure the system is up-to-date and the necessary tools are available
```
$ sudo pacman -Syu
$ sudo pacman -S git base-devel --needed
$ git clone <aur/pkg.git>
$ cd pkg
```
Inspect the source files to verify dependencies, sources, build dependencies. When you are confident this is OK you can proceed to build the package.

```
$ makepkg
```

Resolve build error or missing dependencies if any and install the package

```
$ sudo pacman -U pkg-ver.pkg.tar.xz
```
For a deeper look into possible build issues - read https://forum.manjaro.org/t/howto-fix-a-failing-aur-package-build/104668.

## AUR helpers
Many __CUSTOM scripts__ exist for various reasons e.g.

* the content they provide are proprietary
* the content is not widely use
* drivers for devices commonly used by Windows systems
* deprecated tools e.g. older versions of python

Some has a setup where they find themselves depending on several binary packages not readily available or because a build script has dependencies which is not readily available.

This creates a repetitive task of iterating over the packages - and if there is something a hardcore computer user hates - it is repetitive tasks. They must be scripted.

So a new kind of scripting or application was born - the AUR helpers.

AUR helpers is the kind of tool I would never expect to find in the Arch Linux repositories because an AUR helper automates __CUSTOM packages__.

A.U.R no longer reference Arch User Repository but

__Automated User Response__

1. Consisting of a script/application
2. Which executes a another set of scripts downloaded from a public source
3. The downloaded material is defined by one or more complete strangers
4. The helper abstracts verification
5. The helper executes the script
6. The script fetches sources from third party locations
7. The script executes a series of actions on your personal computer system
8. Which may alter critical system components like kernel or device drivers


## Result

### Worst case scenario
__A completely random example:__
On the next update various system components are updated such as xorg or wayland and because one of the your __CUSTOM packages__ is now incompatible with the system - the system refuses to start - no display manager starts.

You have read about the error __LightDM display manager failed to start__ or __SDDM display manager failed to start__.

### Who are you blaming?

You are blaming the update - where in reality - you should blame your own lack of knowledge or perhaps you just forgot that __CUSTOM package__ which is now dysfunctional and breaks your system.

## What should be learned?

- Exercise due diligence
- Know your system
- Know possible weaknesses
- Remember your __CUSTOM  packages__

Manjaro provides and excellent package helper which is capable of handling various methods of distributing an application - and it can build from __CUSTOM scripts__. 

Although this is indeed a Manjaro feature __CUSTOM packages__ and the Manjaro package manager Pamac makes it possible build from __CUSTOM scripts__ - it is still your system - thus __CUSTOM packages your responsibility__.

## License
Manjaro is a GNU/Linux which is based on [GNU Public License][8] or __GPL__

> 15. Disclaimer of Warranty.
> 
> THERE IS NO WARRANTY FOR THE PROGRAM, TO THE EXTENT PERMITTED BY APPLICABLE LAW. EXCEPT WHEN OTHERWISE STATED IN WRITING THE COPYRIGHT HOLDERS AND/OR OTHER PARTIES PROVIDE THE PROGRAM “AS IS” WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE. THE ENTIRE RISK AS TO THE QUALITY AND PERFORMANCE OF THE PROGRAM IS WITH YOU. SHOULD THE PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF ALL NECESSARY SERVICING, REPAIR OR CORRECTION.
>
> 16. Limitation of Liability.
> 
> IN NO EVENT UNLESS REQUIRED BY APPLICABLE LAW OR AGREED TO IN WRITING WILL ANY COPYRIGHT HOLDER, OR ANY OTHER PARTY WHO MODIFIES AND/OR CONVEYS THE PROGRAM AS PERMITTED ABOVE, BE LIABLE TO YOU FOR DAMAGES, INCLUDING ANY GENERAL, SPECIAL, INCIDENTAL OR CONSEQUENTIAL DAMAGES ARISING OUT OF THE USE OR INABILITY TO USE THE PROGRAM (INCLUDING BUT NOT LIMITED TO LOSS OF DATA OR DATA BEING RENDERED INACCURATE OR LOSSES SUSTAINED BY YOU OR THIRD PARTIES OR A FAILURE OF THE PROGRAM TO OPERATE WITH ANY OTHER PROGRAMS), EVEN IF SUCH HOLDER OR OTHER PARTY HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGES.
>
> 17. Interpretation of Sections 15 and 16.
> 
> If the disclaimer of warranty and limitation of liability provided above cannot be given local legal effect according to their terms, reviewing courts shall apply local law that most closely approximates an absolute waiver of all civil liability in connection with the Program, unless a warranty or assumption of liability accompanies a copy of the Program in return for a fee.
>>  -- [Free Software Foundation - GNU Public License (GPL)][9]


## Disclaimer

>Disclaimer
>
>No warranties, either express or implied, are hereby given for anything provided by Manjaro Linux (“Software”). All Software is supplied without any accompanying guarantee, whether expressly mentioned, implied or tacitly assumed. This information does not include any guarantees regarding quality, does not describe any fair marketable quality, and does not make any claims as to quality guarantees or guarantees regarding the suitability for a special purpose. The user assumes all responsibility for damages resulting from the use of the software. 
>> -- [Manjaro Web - Terms of Use][9]



[1]: https://en.wikipedia.org/wiki/Comparison_of_operating_system_kernels
[2]: https://linuxfromscratch.org
[3]: https://gentoo.org
[4]: https://kernel.org
[5]: https://gitlab.manjaro.org/packages/core
[6]: https://aur.archlinux.org/
[7]: https://wiki.archlinux.org/title/AUR_User_Guidelines
[8]: https://www.gnu.org/licenses/gpl-3.0.html
[9]: https://manjaro.org/terms-of-use/

-------------------------

