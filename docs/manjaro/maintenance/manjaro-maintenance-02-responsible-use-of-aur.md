---
title: 'Responsible use of AUR'
date: '09:59 02-11-2022'
taxonomy:
    category:
        - docs
---

linux-aarhus | 2021-10-16 11:26:19 UTC | Source: [Responsible use of AUR](https://forum.manjaro.org/t/responsible-use-of-aur/86392)

## Be responsible in using the AUR

:warning:  Remember, the AUR is neither officially supported by Arch nor by Manjaro. Using it is at your own risk and your own responsibility.

The AUR ([Arch User Repository](https://wiki.manjaro.org/index.php/Arch_User_Repository)) is like a public library run by volunteers. The books are contributed by the readers and as such we respect both the volunteers and our fellow readers contributing their books for us to read. We all need to be responsible in our use of this repository.

__We are guests allowed to use the library - so let's behave like polite guests.__

With the growing popularity of Arch based distributions - the AUR has become more popular that any Arch user could have ever imagined.

It is difficult to educate on responsible use of a free service - just picture that you were the provider and not aurweb Development Team.

While the AUR technically is just an organized collection of text files containing instructions on how to install certain software whether this is from source code or from binary packages - it contains functionality to search for keywords within package names and their description.

I cannot say what the original intention were - I was not there - but as a developer and former system administrator and small time hosting provider - I can appreciate how the Archlinux users have contributed to the overall ecosystem of Manjaro.

I want to appeal to you - my fellow Manjaro community members - and the anonymous Manjaro user reading this ...

The in-house package manager shipped with Manjaro - Pamac - contains functionality which - when used without thinking - easily contributes to the overload of AUR - which just happened again - today [date=2021-10-14 timezone="Europe/Copenhagen"].

- do not enable AUR unless you strictly need it
- do not enable checking for updates to AUR packages
- do not enable checking for updates to DEV packages

How useful this may be - it also constitutes a lot of useless traffic targeted the AUR search API.

Some time ago the Archlinux Team has considered it necessary to throttle requests to the search api because of user scripts used in conky flooded the search api.

Recently a topic was opened in Manjaro gitlab for Pamac requesting modifications to how Pamac used the search API to minimize the impact on the AUR service.

The incident of today - makes it clear that the AUR web search API is overloaded and I ask of you to consider how you are contributing to the impact on the AUR web.

We all benefit from the availability - so let's not abuse it.

-------------------------
