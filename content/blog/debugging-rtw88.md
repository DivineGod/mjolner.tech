+++
title = "Debugging RTW88"
slug = "debugging-rtw88"
date = "2025-12-15T13:14:00+00:00"
include_in_feeds = true
discoverable = true
is_page = false
+++

These are my quick notes to myself about trying to resolve and issue in the Dogebox OS regarding trying to scan for wifi access points when the driver is already in AP mode as well

This AP+STA mode is supposedly supported by the module installed in the m.2 PCIe slot of the single-board computer. The module is the Realtek RTL8822CE.

## Background
The dogebox and its OS is currently being developed as an open source project by the Dogecoin Foundation. The OS is being build as a simple way to be an apart of the dogecoin network for anyone interested.

## The crash
When installing the OS to the NanoPC-TC we would like to make the experience as simple as possible for anyone to be able to do. We are currently requiring an Ethernet connection to a local lan as configuration and installation and subsequent administration of the system is delivered as a web application.

The friction of needing an Ethernet cable can be an issue for some people and since we already have wifi in the unit we felt that trying to enable an install flow where the dogebox will advertise its own network to the user would lower the barrier of entry.

Getting the dogebox OS to create an AP was as simple as using the `create_AP` package in NixOS. What initially didn’t work was trying to the enable the radio in STA mode at the same time. It. Turned out the version of Linux we were using had an old driver. First I managed to find a pack ported version called `rtw88` and it seemed to work great until at the very end of the configuration part of the install. There, the `dogeboxd`calls out to the Linux system to scan for available networks to allow the user to choose their preferred method of connecting to the internet.

It is in this call that I noticed an error, or rather I noticed the dogebox OS becoming unresponsive and not progressing in the install process.

I decided to upgrade the kernel in an attempt to see if the mainline kernel module would work better.

It’s currently also crashing but the rrror is slightly different. Though I’m not sure how relevant this is. 

One thing I have noticed after about a week of banging my head against this particular problem is that the box isn’t fully broken. I’ve got a keyboard and monitor attached while testing and I ran dmesg to see the error output. When the fault happens this process locks up completely and it seems to me like the system is unresponsive. What I found though, was that the background service running the web application still works and responds to requests made from the browser. 

I’m going to try to ssh into the box and get more information out.

I added a special kernel patches section to the nixos config to add more logging in the hope that it’ll give me an indication of where to look for further progress.

### Narrowing down the issue

Today, Tuesday 16-12-2025, I decided to try running the crashing command without using sudo and to my surprise this worked!

So this led me to file an issue with the rtw88 back-port repository used in kernel we're currently using for the nanopc-t6.

## Creating a bug report

Researching how to debug kernel driver had me stumble across [BUG: Unable to Handle Kernel Paging Request at 00000000f0000000: Debugging PCI Driver Read Error in Linux](https://linuxvox.com/blog/bug-unable-to-handle-kernel-paging-request-at/#3-reproducing-the-error-and-initial-data-collection)

Reading this post helped me with collecting data for my own bug report.

I gathered the `journalctl` output into a file, ran `lspci -vvv` and located the output relating to the Realtek Module rtl8822ce.

## Minimal Reproduction Repository

Tuesday 16-12-2025: I decided to create a [minimal case reproducing](https://github.com/DivineGod/nixos-rtw88-crash) the issue and publish it alongside my notes.

## Bug Report Update - possible fix

Overnight from Tuesday to Wednesday, one of the contributors to the backported rtw88 kernel driver module [responded with a fix](https://github.com/lwfinger/rtw88/commit/f27b6a3b55129f9b7f30742303ffeee479fee8ec).

So it turns out that the culprit might have been trying to access unaligned memory.

## Confirming the fix.

I managed to find a way to use the new fixed backport wireless driver in our dogebox os nixos config.

By making a new module for the rtw88 driver in a sub-directory `rtw88/default.nix` that contains the same source as the nix package, except for the updated commit reference. I could add the following:

```nix
  boot.extraModulePackages =
    let
      rtw88 = config.boot.kernelPackages.callPackage ./rtw88 { };
    in
    [ rtw88 ];
```

I needed to do it this way since we're using a custom version of the kernel and the module will need to know about this version. If not, the version would be the default that nix knows about and we get a compilation error.
