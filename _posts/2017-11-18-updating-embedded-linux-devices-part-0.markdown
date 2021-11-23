---
layout: post
title:  "Updating Embedded Linux Devices: Background"
---


First of, lets set the scene.

You are planing to deploy devices running Linux or already are.

Your embedded Linux devices probably consists of the following components:

-   bootloader
-   Linux kernel and probably a DTB file
-   root-filesystem
-   custom partitions to hold application data
-   custom application

and maybe you have some additional MCUs or FPGAs connected to your SoC that is
running Linux which also has an firmware which needs to be updated.

Once these devices are in production (in the wild) you must have some form of
mechanism to update them. If your devices have networking connectivity and are
connected to the Internet you have the possibility to do Over The Air (OTA)
updates, if not you must provide other means of doing software updates.

#### Why?

Before we go any further lets talk a bit about why we need software updates
throughout a product life-cycle.

-   We as software developers do not write perfect code and as our systems get
    more complex it is harder to get full test coverage. We need a way to
    mitigate bugs when devices have left the building.

-   Feature growth. We are often in a race to bring products to the market and
    if we are able to deliver “base functionality” in a initial release and
    provide more features later on by doing software updates our chances
    increase of winning that race.
    
-   Security! Your embedded Linux devices contains millions lines of code on
    which you rely on to keep your device secure, preventing third parties from
    accessing your device to extract proprietary code or do other malicious
    things. Over time exploits are exposed in both the Linux kernel and maybe
    other components that you use in you system. These exploits are often
    publicly known after a certain time and are easily exploited. Therefore
    continues security updates are needed if you want to protect your device.
    [CVE](https://cve.mitre.org/about/) keeps a database and assigns an unique
    CVE ID to each known vulnerability and status of fixes.

Above is not revolutionary at all and most of you reading this already know of
these fundamental requirements for your devices. But what my experience has
showed me is that above requirements are usually solved at the end of a product
development cycle, the idea that it should be solved probably has been there
from the beginning but somehow it is not prioritized and this has consequences
that I will talk about later on.

#### What?

Lets define some key requirements that we would like in a software update
solution.

1.  It must be able to update our application, and Linux kernel and other
    low-level components

2.  It must not under any circumstances break (“brick”) our devices.
    [Here is one example of what can happen if this requirement is not fulfilled](https://techcrunch.com/2017/08/14/wifi-disabled/)

3.  It must be **atomic**, update succeeded or update failed. Nothing in-between
    that could result in that the device still “functions” but with undefined
    behavior.

4.  It must be able to install **signed** updates, prevents third parties from
    installing software on our device.

5.  OTA updates must be performed trough an secure communication channel.

Simple enough, but not really. It is hard to cover all the above and anyone
that has tried can vouch for that.

Projects that do not implement a software update solution at the beginning tend
to do it in the end of the development cycle of a product, usually in a hurry
do ship devices and the software update solution becomes an “after thought”.
This normally results in writing a “homegrown” solution that that only covers
requirement Nr.1, and maybe only able to update the application with no way of
updating Linux kernel and other critical parts of the system.

It is crucial that above requirements are resolved during the first phases of a
project, and this usually means that you can use the solution during the full
development cycle gaining confidence in its stability.

#### How?

If you are considering writing your own solution, please do not!

There are now some pretty awesome well established open-source solutions
available to us that take care of all or some of the complexity.

Here is a list:

-   [swupdate](https://github.com/sbabic/swupdate)
-   [rauc](https://github.com/rauc/rauc)
-   [Mender](https://mender.io/)
-   [OSTree/libostree](https://ostree.readthedocs.io/en/latest/)
-   [aktualizr](https://github.com/advancedtelematic/aktualizr), uses libostree.
-   [swupd](https://github.com/clearlinux/swupd-client)

I do not mention the “standard” package managers which you could probably use
to some extent, but they do have the problem of not being “atomic” and generally
result in a dependency nightmare and hard to test. Convince me otherwise!

#### Ending

This post is the first one in a series where I will examine the different
solutions available to us (probably not all of them).

I will do an analysis of what core features they are offering and I will get my
hands dirty and try them out on some hardware (Beaglebone Black and
Raspberry Pi 3 seems to be a reference board for many of them) and share my
observations here with a tutorial on how you could test them as well.

First up is [swupdate](https://github.com/sbabic/swupdate). Stay tuned!
