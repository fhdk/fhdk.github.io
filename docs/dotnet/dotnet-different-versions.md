---
title: 'Different dotnet versions'
date: '07:57 15-11-2024'
taxonomy:
    category:
        - docs
---

Source [https://aur.archlinux.org/packages/dotnet-host-bin](https://aur.archlinux.org/packages/dotnet-host-bin)

IMPORTANT INSTALLATION INFO (a reminder for myself as well):

For dotnet to work you need to EXPLICITLY install:

    ONE dotnet-host - highest version possible
    ANY NUMBER of dotnet-runtimes (and its sdks after if you want to build as well)

If you keep the install order in mind and you don't rely on pacman to resolve your dependencies you will be fine.

Longer explanation:

Every dotnet-sdk is dependent on a specific version of dotnet-runtime, this is built into dotnet.

Technically you only need the latest dotnet-sdk because it can build to any earlier versions.
