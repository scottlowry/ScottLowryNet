---
title: "Saving a Surface Pro 3 from E-waste Using Ubuntu"
date: 2025-06-11
draft: true
summary: "Use Ubuntu to resurrect an old Surface Pro 3 from the grave."
tags: ["Linux", "Ubuntu", "Microsoft Surface", "linux-surface"]
---

# Introduction

A few years ago my wife upgraded from a Surface Pro 3 to a new Lenovo Yoga. She hung on to the Surface and eventually gave it me to sell. It's too old to run Windows 11 and Windows 10 doesn't run well on it now either. It's not worth selling so I decided to recycle it. That's until I came across this project: **[linux-surface](https://github.com/linux-surface/linux-surface)**. It's a special build of the Linux Kernel specifically for Surface devices. Lucky for me, the Surface Pro 3 is fully supported unlike some other Surface devices. 

## Installation

Installing was really easy. I started off doing a typical Ubuntu install by flashing an ISO to a USB thumbdrive and booting it. I used Balena Etcher on my Mac to do it but there's other tools like Rufus for Windows. I should note that I chose to use Ubuntu 22.04.5 LTS. The documentation for **linux-surface** doesn't state which version to use except mentioning the package `iptsd` for multitouch support requires at least 22.04. That may only apply to Surface Pro 4 or newer from what I can tell though. I chose 22.04.5 instead of something newer because of better compatibility and less overhead for older hardware. This Surface Pro 3 has a 4th gen i5 dual-core, quad-thread CPU with 8GB DDR3 memory so the less bloat the better. 

After installing Ubuntu and running updates, it was easy to install **linux-surface**. I followed their [Installation and Setup](https://github.com/linux-surface/linux-surface/wiki/Installation-and-Setup) guide to add their package repo to `apt` and install the specialized kernel and dependencies. I also installed the Secure Boot keys for the kernel and updated GRUB. 

## Performance

I was nervous during the install as the Ubuntu install menu was really laggy and the fans in the Surface were going full blast. It didn't take long to install though and booted up quickly after finishing. In fact, it's pretty snappy. It boots up in just a few seconds, apps open quickly, windows move around without lag, and fans haven't been loud. Even the touch support and screen rotation work perfectly. I'm really impressed. 

## Final Thoughts

Even though Ubuntu runs great on it, I still don't see it as worth selling. Most people would want to replace Ubuntu with Windows which then makes it borderline useless. I plan to use it for the following:
- Remote monitoring of my 3D printer upstairs from downstairs
- Display recipes and YouTube cooking videos in the kitchen