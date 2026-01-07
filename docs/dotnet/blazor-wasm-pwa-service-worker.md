---
title: 'Blazor WASM PWA service-worker'
date: '05:26 22-04-2025'
taxonomy:
    category:
        - docs
---

## Update notifier 

Using a sw-registrator.js to expose service-worker.js and an update notification to be consumed by the Blazor WASM PWA.

Drawing attention to outdated versions of a deployed web application is cumbersome and explaing the enduser they need to close all tabs and windows related to the app is equally cumbersome, and they tend to forget in heartbeat.

Lucky for me - others have had the same issue - I found a nice workaround on [Microsoft Tech Community] with a reference to the original author [Wouter Huysentruit] and I am keeping them here to support my memory (flash memory, the kind that is gone in a flash)

[Microsoft Tech Community]: https://techcommunity.microsoft.com/discussions/web-dev/blazor-wasm-pwa-%E2%80%93-applications-updates-cache-busting-with-notification-or-force-/3920976

[Wouter Huysentruit]:https://whuysentruit.medium.com/blazor-wasm-pwa-adding-a-new-update-available-notification-d9f65c4ad13