---
layout: post
title:  "Updating Embedded Linux Devices: Update strategies"
---

This post if part of the “Updating embedded Linux devices” series, previous
posts are:

- [Updating Embedded Linux Devices: Background](https://mkrak.org/2017/11/18/updating-embedded-linux-devices-part-0/)

Before I start talking about different projects I wanted to write a bit about
common update strategies on embedded Linux systems unrelated to any specific
project. By reading this it will be easier to follow my coming articles because
each project has chosen one or more strategies to focus on and I wanted to write
down the pros and cons unrelated to any project.

## The Desktop way

That is package managers (dpkg, rpm etc.). It is quite common that this
question arises. It works on desktop why can I not use this for my embedded
device? Well you can, but I would not choose to go down this path voluntarily.
Desktop package managers are not designed with embedded device uses-cases in
mind. And it can only go downhill from here :).

Package managers do not perform atomic updates. And this is one of the core
requirements for a robust software update solution. One power-loss in an update
sequence and you have introduced “weirdness” in to your device from which you
will never recover from.

Using package managers it is hard to “mirror” your release across thousands of
devices, which is easily accomplished with image based updates. And if you are
not able to mirror your release to all you devices you are in a
testing/configuration hell.

Even Fedora, Redhat and CentOS have realized this, as they now provide an
Atomic Host distribution which is hybrid of image based updates with rpm
([rpm-ostree](https://rpm-ostree.readthedocs.io/en/latest/)).

From [Fedora Atomic Host site](https://getfedora.org/en/atomic/):

>Atomically update your system from the latest upstream OStree. Make your
servers identical, and easily roll back if there’s an upgrade problem

Which basically confirms what I just wrote, that ordinary package managers are
not reliable enough to accomplish the above which is a critical requirement on
embedded devices.

## The Image way

There is one draw back with image based updates and that is actuall size of the
update image, and this of course because we always do full OS upgrades. The
image size can typical be reduced using compression but it will still be of a
significant size. This is usually only a problem when you perform OTA updates
using a mobile connection (GPRS, 3G, LTE etc.).

### Single Copy

You would probably deploy software updates using this method if you have a
device with very limited storage space which does not allow you to run a
“Dual Copy” strategy (more on this below).

The partitioning of your device could look like this when running a
“Single Copy” strategy:

![](/assets/images/single-copy-update.png)

You normally have an “hard-coded” area where you put your boot-loader and this
is something that is decided by the SoC. Nothing odd here.

When running a “Single Copy” update strategy you rely on a “Recovery OS”, which
is normally a minimal Linux kernel and a minimal root-filesystem, just enough
features to allow the device to boot and perform updates. The root-filesystem
is loaded in to device memory and mounted as a `tmpfs`. Booting using only RAM
is necessary if you are to write updates to your “normal” storage medium. This
boot concept is often referred to as an `inird` or `initial ramdisk` and is
widely used by
[desktop Linux distributions](https://en.wikipedia.org/wiki/Initial_ramdisk).

It is fairly common to embed a cpio archive containing the root-filesystem in
the Linux kernel binary using `CONFIG_INITRAMFS_SOURCE` which will result in a
single file to be stored on our storage medium and this is what the
“Recovery OS” represents.

A typical OTA update using a “Single Copy” strategy contains the following
steps (security checks excluded for simplicity):

- Device performs regular check in with OTA servers and is notified of the
  availability of an update

- Update downloads to data partition. User is prompted to install the update.

- Device reboots into “Recovery OS”, that is it loads the recovery ramdisk image
  and boots using RAM only.

- Data is pulled from the update image located on the data partition and used
  to update the system.

- Device reboots normally.

- The newly updated boot partition is loaded, and it mounts and starts executing
  binaries in the newly updated “Regular OS”.

Does above look familiar? If you have ever updated Android version on your
phone, it should. Because this strategy has been used by Android based phones
for a long time and it probably still is. Note that it is only used for the
underlaying parts of an Android operating system. User application are
untouched when performing an Android update.

Support for a “Dual Copy” strategy was added in Android N release calling it
“Seamless System Updates”. Though they only added support for it as option
for the manufacturers to choose from which means that a lot phones still use
the old strategy (“Single Copy”). It is also not recommended to use the
“Dual Copy” strategy on Android phones with 4GiB or less available storage
space (according to their developer resources).

There are some obvious down-sides with using a “Single Copy” strategy:

- Device is unusable while performing an update, depending on size of the
  update of course but this could take minutes. Android users know what I am
  talking about.

- You need to reserve storage space to store an update image while you reboot
  in to the “Recovery OS”

- There is no fall-back, that is if the software we installed is faulty or
  installed with errors or it was interrupted we will end up with a device that
  does not boot to “Regular OS” but instead only boots to “Recovery OS” in which
  it hopefully can retry the upgrade but if the new software is faulty a retry
  wont help and user interaction is necessary.

There are some up-sides as well:

- Compared to a “Dual Copy” strategy it consumes less storage space, and you
  if you are really tight on storage then this might be the only option that is
  viable.

- It is battle-tested in millions of Android devices which of course is a
  confidence builder if you are thinking about deploying a similar strategy on
  your device.

### Dual Copy

Luckily for us storage medium are cheap and now days it is not uncommon to
have GiB’s of available instead of a few MiB’s on our embedded devices.
And this is what really opens up for a a “Dual Copy” strategy.

A “Dual Copy” strategy can be partitioned like this:

![](/assets/images/dual-copy-update.png)

Again boot-bootloader at the “hard-coded” location.

Then we have the interesting part. In a “Dual Copy” strategy we have two sets
of “Regular OS” a.k.a root file-systems. One is active which is the one we have
booted from, the other one is inactive.

A typical OTA update using a “Dual Copy” strategy contains the following steps
(security checks excluded for simplicity):

- Device performs regular check in with OTA servers and is notified of the
  availability of an update.

- Update downloads/streams to inactive “Regular OS” part.

- Device reboots and the active and inactive “Regular OS” parts shift places,
  meaning that we are now booting our updated part which was previously inactive
  and the active one is marked as inactive and this is where sequential updates
  will be written.

And that is it. Simple really.

Compared to a “Single Copy” strategy:

- We no longer need to reserve space to put an update image as it can be
  streamed straight to the storage medium (the inactive part), but we do take
  up more space still as we must keep two copies of the “Regular OS”.

- We do not need to prompt the user for anything as this can be done in the
  background and on the next boot you will boot a updated OS. Though in some
  situations you can not have the update done completely seamless and there
  needs to be a confirmation from a user or an application that it is OK to
  proceed.

- No downtime beside a normal device reboot.

- We now have the possibility do to an roll-back, that is if the new software
  does not start-up or does not work as expected we can always revert back to
  the previous image. The device remains functional and update can be retried
  again.

If we are to take Android as a example or reference again, it is clear that it
is in the process of migrating to a “Dual Copy” strategy. At least on devices
that have the available size for it. The Pixel and Pixel XL phones ship with
this functionality and as I have mentioned earlier they market it as
“Seamless Software Updates”. All Chromebooks use this strategy as well. Which
again proves that it is a battle-tested strategy that can work on a large scale.

### The differential way

There is one type of image update that does not suffer from the “size problem”
and that is using binary deltas on images and downloading only the diff to the
device when performing an update.

But I will not go in to this much further here as I will do an in depth post
covering this topic later on.

## The U-boot way

That is to use U-boot or any other boot-loader as an update client.

This concept has been a somewhat popular method to update embedded Linux
devices. I have been down this road and hopefully I will not repeat it :).

It is deceitfully easy to implement an update solution in U-boot by writing a
small script and then prepare it with `mkimage`, and make sure that U-boot
probes for a specific script name on boot. What this script normally does is
to load an image or multiple images from an USB flash drive or even from rootfs
and then writes the images to certain parts of the storage medium. By loading
the update image from rootfs we could probably extend this update strategy with
OTA capability, because we can not really use the networking stack in U-boot to
do anything securely.

Here is a simplified example of such logic and script:

```bash
"if fatload usb 0:1 ${loadaddr} ${updatefilename}; then " \
 "source ${loadaddr}; " \
"fi; "
```

And the script content can look something like this.

```bash
# Update Linux kernel
fatload usb 0:1 ${loadaddr} uImage;
nand erase.part kernel;
nand write ${loadaddr} kernel ${filesize};
```

```bash
# Update rootfs
fatload usb 0:1 ${loadaddr} ubi.img;
ubi write ${loadaddr} rootfs ${filesize};
```

One could also implement some security features such as signed update by
wrapping your script and image in the
[FIT](http://git.denx.de/?p=u-boot.git;a=blob_plain;f=doc/uImage.FIT/signature.txt;hb=HEAD)
format making sure that only signed update images are loaded.

Easypeasy right?

My example above is a simplification and even though it might work to some
degree there is a lot of gaps and corner cases that need to be handled if we
are to create a robust update solution. Which means that we need to increase
the complexity in a very limited environment.

U-boot is after all only a boot-loader which is intended to load your software.
And here in is the problem with this solution. Because U-boot is only a
boot-loader it does not have all the latest security fixes and the drivers in
U-boot does not always contain the latest features nor bug fixes, nor does it
contain all drivers for your hardware as it does not make sense to support
everything in a boot-loader. Most drivers in U-boot are forks of drivers from
the Linux kernel.

In my simple example above we use a huge amount of U-boot code with only a few
lines. To summarize:

- USB software stack (that also includes the hardware driver for the specific
  SoC and Host side USB Mass storage class)

- DOS filesystem (fatload)

- NAND hardware driver

- UBI, and we probably need to use UBIFS if we are to load an update image from
  the rootfs

And believe me this is just asking for trouble. You will probably end up
snail-mailing USB flash drives (tested to work with your U-boot) with software
updates because none of the customers flash drivers work in U-boot. Is this
also OTA firmware updates? :).

The shortcomings are simply to many for this to be a viable method to do
firmware updates on a larger scale in a production environment.

## Summary

There are many ways to accomplish something with similar end result and it
should be carefully assessed what strategy fits your current project.

Moving forward I will start writing articles about the existing projects that
I mentioned in my previous
[post](https://mkrak.org/2017/11/18/updating-embedded-linux-devices-part-0/)
and then we can look back on to this article and compare with the
implementations.

I would like to extend a thank you to my employer
[Endian Technologies AB](https://endian.se/) who has allowed me to work on this
during work hours.

Here are some additional resources regarding update strategies:

- [Software updates for embedded Linux: requirement and reality by Chris Simmons](https://mender.io/resources/guides-and-whitepapers/_resources/Software%2520Updates.pdf)
- [SWUpdate documentation: Software Management on embedded systems](https://sbabic.github.io/swupdate/overview.html)
- [Android A/B (Seamless) System Updates](https://source.android.com/devices/tech/ota/ab/)
- [Android Non-A/B System Updates](https://source.android.com/devices/tech/ota/nonab/)
