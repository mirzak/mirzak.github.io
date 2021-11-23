---
layout: post
title:  "Updating Embedded Linux Devices: SWUpdate"
---

This post if part of the “Updating embedded Linux devices” series, previous
posts are:

- [Updating Embedded Linux Devices: Background](https://mkrak.org/2017/11/18/updating-embedded-linux-devices-part-0/)
- [Updating Embedded Linux Devices: Update strategies](https://mkrak.org/2018/01/10/updating-embedded-linux-devices-part1/)

First up on my software update journey is:

In 2014 I attended ELCE (Embedded Linux Conference Europe) in Düsseldorf,
Germany, and this is also the time and place when I was introduced to
[SWUpdate](https://github.com/sbabic/swupdate) when Stefane Babic did a
[talk](https://lccoelce14.sched.com/event/1yFSFUQ) about this new project that
 he had started withing DENX. Was a great presentation and I left that session
 with a lot of new insights on the topic.

SWupdate has since then grown in to an mature project with an active community,
I try to keep up on what is happening in the project on the
[mailing list](https://groups.google.com/forum/#!forum/swupdate) but it is
getting tougher to keep up as there is much going on. At the time of me writing
this there has been 16 releases and now days a new release is done every
3-4 months, a very healthy release cycle in my opinion.

But what is SWUpdate?

SWUpdate is an highly configurable and feature rich update agent
(written mostly in C). It does not enforce one specific kind of usage
(strategy) but instead provides a set of tools giving one the possibility to
fit it to your specific needs. In other words it is an framework to enable
software updates on you embedded Linux device.

Key features:

- flexible update image format (.swu)
    - Syntax is one of
        - libconfig (default)
        - JSON
        - XML with an Lua parser
    - Supports targeting multiple targets/revisions with the same image file
- Supports the common storage mediums trough different handlers
    - eMMC/SD,
    - Raw flash (NAND, NOR, SPI-NOR)
    - UBI volumes
    - extensible with custom handlers in C or in Lua
- Extensible with OTA support
    - Relies on [Hawkbit](https://projects.eclipse.org/proposals/hawkbit) for the server part
- Webserver can be activated to upload updates to device
- IPC trough a domain socket to get progress information on update
- Support for encrypted/signed update images
- Supports a master/slave approach, master can update multiple slaves running SWUpdate

There is a lot more features, and you can read more about them
[here](http://sbabic.github.io/swupdate/swupdate.html)

The most common usage of SWUpdate is to do image based updates using a
“Single Copy” or a “Dual Copy” strategy. SWUpdate does support updating single
files on you root file-system or other partitions but I would assume that this
is only used in some specific cases such as updating some configuration file on
a “data” partition.

Lets get our hands dirty. But wait, what are the requirements of running
SWUpdate. I do not mean the dependencies to compile the program but the
requirements on the platform to be able to perform image based updates. Well
first of you must use a storage medium which SWUpdate supports and there exists
a handler for it, otherwise you need to write one but you are pretty safe here
as SWUpdate supports the most common mediums used on a embedded device.
Secondly you need a boot-loader which will have handle the “Dual Copy”
switching or the entrance to the “Recovery OS” in a “Single Copy” mode. This is
normally done by modifying some variable that both Linux and the boot-loader
have access to (environment variables). SWUpdate currently supports U-boot and
GRUB as boot-loaders.

I will be using a Beablebone Black as reference board which ticks of the
requirements and I was thinking of doing the following as a first step:

- Install the latest official debian image on by board
- Download and compile SWUpdate natively on BBB (to keep it simple)
- Perform a “raw file” update, that is we will update an single file

This is probably not something that I recommend for my readers to do as this is
a quite manual process and I am only doing this to provide some insight in the
different steps of doing an update with SWUpdate on a “concept” level. This is
probably not something that you can re-use a production environment.

I will later on do the same thing using Yocto which is a more production ready
approach.

Lets flash an SD card with the latest BBB debian image (at the time of me
writing this the latest image was 2017-10-10, this could have changed but
should work with newer images as well):

```bash
$ wget https://debian.beagleboard.org/images/bone-debian-9.2-iot-armhf-2017-10-10-4gb.img.xz
$ unxz bone-debian-9.2-iot-armhf-2017-10-10-4gb.img.xz
$ sudo dd if=bone-debian-9.2-iot-armhf-2017-10-10-4gb.img of=/dev/mmcblk0
```
Insert the card and boot it.

```bash
Debian GNU/Linux 9 beaglebone ttyS0

BeagleBoard.org Debian Image 2017-10-10

Support/FAQ: http://elinux.org/Beagleboard:BeagleBoneBlack_Debian

default username:password is [debian:temppwd]

beaglebone login:
```

Before we can proceed we need to install some extra packages on our “fresh” image that are needed by SWUpdate:

```bash
debian@beaglebone:~$ sudo apt-get install lua5.2-dev libssl-dev libconfig-dev libarchive-dev \
    libzmq3-dev libz-dev libcurl4-gnutls-dev libjson-c-dev
```

Now we download the SWUpdate source (2017.11 was the latest release at the time
of writing this):

```bash
debian@beaglebone:~$ git clone https://github.com/sbabic/swupdate.git -b 2017.11 && cd swupdate
```

SWUpdate uses the same build system as the Linux kernel (Kbuild) which might be
familiar to some. This means that there are a set of pre-defined configuration
files that we can use to configure and build SWUpdate.

```bash
debian@beaglebone:~/swupdate$ ls configs/
cms1_defconfig
cms_defconfig
debian_defconfig
hawkbit_defconfig
nodwl_defconfig
raspi_defconfig
sha256_defconfig
swuforwarder_defconfig
test_defconfig
with_lua_handlers_defconfig
with_lua_nohandlers_defconfig
without_libconfig_defconfig
without_lua_defconfig
with_systemd_defconfig
```

We are going to use `test_defconfig` which is convenient for our test purpose.
Some key features that are enabled/disabled.

- No bootloader support (we do not want this feature at this stage as this
  requires some integration work on the board)
- Webserver enabled
- Signed/Encrypted images
- Support to download update image from URL

Lets configure our project:

```bash
debian@beaglebone:~/swupdate$ make test_defconfig
```

And we can use `menuconfig` to browse/change the configuration of our project

```bash
debian@beaglebone:~/swupdate$ make menuconfig
```

![](/assets/images/swupdate-menuconfig.png)

But we are happy with the options enabled in `test_defconfig` so we are not
gonna make any additional changes.

Lets build and install:

```bash
debian@beaglebone:~/swupdate$ make && sudo make install
```

If we try and start the binary that we have compiled we get the following error:

```bash
debian@beaglebone:~$ swupdate
Swupdate v2017.11.0
Licensed under GPLv2. See source distribution for detailed copyright notices.
swupdate built for signed image, provide a public key file

< ... \>
```

Since we enabled support for signed images we must provide a public key and all
update images must be signed with the corresponding private key.

Lets create a key-pair, we will do it on the BBB for convenience:

```bash
# Create private key
debian@beaglebone:~$ openssl genrsa -out swupdate-priv.pem

# Create a public key
debian@beaglebone:~$ openssl rsa -in swupdate-priv.pem -out swupdate-public.pem \
 -outform PEM -pubout
```

And now we can start SWUpdate:

```bash
debian@beaglebone:~$ sudo swupdate -v -k /home/debian/swupdate-public.pem \
 -w "-document_root /home/debian/swupdate/www"
```

Since we built SWUpdate with the web-server enabled we can now go to our
browser and go to `http://<device ip>:8080/`, and we will be presented with
this:

![](/assets/images/swupdate-webserver.png)

Very simple web-interface that allows us to upload an update image, and its
power is really in that it is so simple and you need it to be. If you have
devices in the “wild” and can not provide updates over-the-air, you will rely
on people that are maybe not as technologically advanced as you are to update
devices, and then simplicity is key!

We have not yet created a update image so we can not upload anything, or well I
tried uploading some random file trying to fool it but as expected it did not
like that very much:

```bash
[network_initializer] : Main loop Daemon
Waiting for requests...
[network_initializer] : Main thread sleep again !
Image invalid or corrupted. Not installing ...
ERROR core/cpio_utils.c : extract_cpio_header : 316 : CPIO Header corrupted, cannot be parsed
ERROR core/cpio_utils.c : get_cpiohdr : 44 : CPIO Format not recognized: magic not found
Software Update started !
File information: random-file.zip size: 364724005 bytes
```

So lets try and construct a valid update image instead. First we must create
a `sw-description` file, which shall contain a set of instruction to SWUpdate
(what to install/update and where). The default syntax for this file is based
on [libconfig](http://www.hyperrealm.com/libconfig/libconfig_manual.html) and
it looks like this in our simple case:

```bash
software =

{
 version = "1.0.1";
 beaglebone = {
     hardware-compatibility: [ "1.0" ];
         files: (
             {
             filename = "SWUpdate";
             path = "/SWUpdate";
             sha256 = "27ebafbe58603437d819cde3470811e5e64a8aa876a4fa680e5266e36bbee519";
             }
         );
     };
}
```
We want to update/install the file `SWUpdate` and the location on the target is
`/SWUpdate`. Simple enough. The `sw-description` files are highly configurable
and I will not try and to cover every aspect of it here as it already documented
[here](https://sbabic.github.io/swupdate/sw-description.html).

Our `sw-description` file says that it is only compatible with the hardware
version `1.0` of the `beaglebone` board. SWUpdate parses `/etc/hwrevision` on
start-up to figure out which board it is running and what version. We must
provide this file and it must contain the same information that we have provided
in our `sw-description` file:

```bash
echo "beaglebone 1.0" > /etc/hwrevision
```

And we must restart SWUpdate because it only parses this on start-up.

We must also create the `SWUpdate` file that we want install/update and we
should end up with this:

```bash
root@beaglebone:~/update-image# ls
sw-description SWUpdate
root@beaglebone:~/update-image# cat sw-description
software =
{
 version = "1.0.1";
 beaglebone = {
     hardware-compatibility: [ "1.0" ];
         files: (
             {
             filename = "SWUpdate";
             path = "/SWUpdate";
             sha256 = "27ebafbe58603437d819cde3470811e5e64a8aa876a4fa680e5266e36bbee519";
             }
         );
     };
}

root@beaglebone:~/update-image# cat SWUpdate
SWUpdate v1
```

The format of the update image that SWUpdate expects is an `cpio` archive and
the `sw-description` file must be the first file in the archive. To achieve this
we can run the following:

```bash
root@beaglebone:~/update-image# export FILES="sw-description SWUpdate"
root@beaglebone:~/update-image# for i in $FILES; do echo $i; done | cpio -ov -H crc > update-image-v1.swu
sw-description
SWUpdate
3 blocks
root@beaglebone:~/update-image# ls
sw-description SWUpdate update-image-v1.swu
```

And we have just created our first SWUpdate update image file! Lets try and upload it trough the web-interface:

(pasting log from web-interface instead of screen-shoots from now on)

```bash
[network_initializer] : Main loop Daemon
Waiting for requests...
[network_initializer] : Main thread sleep again !
Image invalid or corrupted. Not installing ...
[extract_file_to_tmp] : description file name not the first of the list: SWUpdate instead of sw-description.sig
[extract_file_to_tmp] : Found file: filename sw-description size 243
Software Update started !
File information: update-image-v1.swu size: 1024 bytes
```

It is complaining that our image is corrupted or invalid. It is not super
obvious what is wrong here, but I intentionally uploaded an image that is not
signed and was expecting it to fail. Since we built SWUpdate with “signed image”
support it will not accept images that are not signed with a private key that
matches the public key we provided when we started SWUpdate.

So lets try this again. To add a signature to our update image we must create
the `sw-description.sig` file and it needs to be placed in the `cpio` archive
directly after the `sw-description` file.

To create an signature we must run the following:

```bash
root@beaglebone:~/update-image# openssl dgst -sha256 -sign \
    ~/swupdate-priv.pem sw-description > sw-description.sig
```

And re-create the cpio archive:

```bash
root@beaglebone:~/update-image# export FILES="sw-description sw-description.sig SWUpdate"
root@beaglebone:~/update-image# for i in $FILES; do echo $i; done | cpio -ov -H crc > update-image-v1.swu
sw-description
sw-description.sig
SWUpdate
3 blocks
```
And upload the file again:

```bash
[network_initializer] : Main loop Daemon
Waiting for requests...
[network_initializer] : Main thread sleep again !
SWUPDATE successful !
[install_raw_file] : Installing file SWUpdate on /SWUpdate
[install_single_image] : Found installer for stream SWUpdate rawfile
Installation in progress
[network_initializer] : Valid image found: copying to FLASH
[extract_files] : Found file: filename SWUpdate size 12 required
[check_hw_compatibility] : Hardware compatibility verified
[check_hw_compatibility] : Hardware beaglebone Revision: 1.0
[swupdate_verify_file] : Verified OK
[swupdate_verify_file] : Verify signed image: Read 332 bytes
[parse_files] : Found File : SWUpdate --> /SWUpdate (ROOTFS)
[parse_hw_compatibility] : Accepted Hw Revision : 1.0
[parse_cfg] : Version 1.0.1
[extract_file_to_tmp] : Found file: filename sw-description.sig size 256
[extract_file_to_tmp] : Found file: filename sw-description size 332
Software Update started !
File information: update-image-v1.swu size: 1536 bytes
```

Success! And we can see that the file got installed:

```bash
root@beaglebone:~# cat /SWUpdate
SWUpdate v1
```

SWUpdate also exposes an API for third party applications to monitor progress
of an update and provides some control as well of the process. A client library
is provided which communicates trough an Unix Domain Socket with the SWUpdate
process.

There are a couple of reference applications that was built together with
SWUpdate that utilize the client library.

```bash
debian@beaglebone:~/swupdate/tools$ ./progress --help

./progress (compiled Jan 21 2018)

Usage ./progress [OPTION]

-c, --color : Use colors to show results
-r, --reboot : reboot after a successful update
-w, --wait : wait for a connection with SWUpdate
-p, --psplash : send info to the psplash process
-s, --socket <path> : path to progress IPC socket
-h, --help : print this help and exit
```


And this is one example what it can look like when it is running while
performing an update:

```bash
debian@beaglebone:~/swupdate/tools$ ./progress
Trying to connect to SWUpdate...
Connected to SWUpdate via /tmp/swupdateprog
Update started !
Interface: UNKNOWN

[ ============================================================ ] 1 of 1 100% (SWUpdate)

SUCCESS !
```

Above could be used to display information about the update process if your
device has a graphical interface. One can also start or stop an update trough
the client library API, meaning that the application can be in full control
when the update can be started.

Moving forward my intention is to do a more advanced setup using SWupdate. For
this we will utilize Yocto where SWUpdate is supported with the
[meta-swupdate](https://github.com/sbabic/meta-swupdate) layer. There is also a
layer provided a reference implementation of “Dual Copy” strategy on BBB and
Raspberry Pi 3 boards,
[meta-swupdate-boards](https://github.com/sbabic/meta-swupdate-boards).

For this we must setup our Yocto environment:

```bash
$ mkdir swupdate-yocto && cd swupdate-yocto

swupdate-yocto$ mkdir layers && cd layers
swupdate-yocto/layers$ git clone git://git.yoctoproject.org/poky -b rocko
swupdate-yocto/layers$ git clone git://github.com/openembedded/meta-openembedded.git -b rocko
swupdate-yocto/layers$ git clone https://github.com/sbabic/meta-swupdate -b rocko
swupdate-yocto/layers$ git clone https://github.com/sbabic/meta-swupdate-boards.git -b master
swupdate-yocto/layers$ git clone https://github.com/agherzan/meta-raspberrypi.git -b rocko
swupdate-yocto/layers$ cd ..
swupdate-yocto$ . layers/poky/oe-init-build-env build
swupdate-yocto/build$ bitbake-layers add-layer ../layers/meta-openembedded/meta-oe
swupdate-yocto/build$ bitbake-layers add-layer ../layers/meta-openembedded/meta-multimedia
swupdate-yocto/build$ bitbake-layers add-layer ../layers/meta-openembedded/meta-python
swupdate-yocto/build$ bitbake-layers add-layer ../layers/meta-openembedded/meta-networking
swupdate-yocto/build$ bitbake-layers add-layer ../layers/meta-raspberrypi
swupdate-yocto/build$ bitbake-layers add-layer ../layers/meta-swupdate
swupdate-yocto/build$ bitbake-layers add-layer ../layers/meta-swupdate-boards

```

And now we can build an image:

```bash
swupdate-yocto/build$ MACHINE=beaglebone bitbake update-image
```

or

```bash
swupdate-yocto/build$ MACHINE=raspberrypi3 bitbake update-image
```

I have used `beaglebone` as my target and that is what I will refer to from now
on.

Time for a coffee!

Hopefully the build finished successfully and there are two images that are of
interest to us:

- `core-image-full-cmdline-beaglebone.wic` – Image to write to the eMMC on the
  BBB, which will contain SWUpdate which is also started at boot-up. We have to
  utilize the eMMC because the boot partition logic is setup to utilize this.

- `update-image-beaglebone.swu` – This is the update image we will provide to
  SWUpdate, and it contains the full root-filesystem

We transfer the `core-image-full-cmdline-beaglebone.wic` image to our BBB that
is running the debian image that we used earlier.

Write the image to the eMMC:

```bash
debian@beaglebone:~$ sudo dd if=core-image-full-cmdline-beaglebone.wic of=/dev/mmcblk1 bs=1M
```

Eject the uSD card, to make sure that it does not boot from there anymore and
restart the device (power-cycle).

Once we have booted from the eMMC we can see that SWUpdate is running on our
image already:

```bash
root@beaglebone:~# ps ax | grep swupdate

 503 ? S 0:00 /usr/bin/swupdate-progress -w -p -r
 504 ? Sl 0:00 /usr/bin/swupdate -v -H beaglebone 1.0 -e stable,copy2 -f /etc/swupdate.cfg -u -w
 530 ? Sl 0:00 /usr/bin/swupdate -v -H beaglebone 1.0 -e stable,copy2 -f /etc/swupdate.cfg -u -w
 531 ? S 0:00 /usr/bin/swupdate -v -H beaglebone 1.0 -e stable,copy2 -f /etc/swupdate.cfg -u -w
```

One interesting take away from above is that there is an `-H` option to provide
board and hardware revision, which we earlier provided trough the
`/etc/hwrevision` file.

Also one interesting thing that might seem confusing is that if we look at how
our eMMC is partitioned:

```bash
root@beaglebone:~# fdisk -l /dev/mmcblk1

Disk /dev/mmcblk1: 1.8 GiB, 1920991232 bytes, 3751936 sectors

Units: sectors of 1 * 512 = 512 bytes

Sector size (logical/physical): 512 bytes / 512 bytes

I/O size (minimum/optimal): 512 bytes / 512 bytes

Disklabel type: dos

Disk identifier: 0x9bf174dc

Device Boot Start End Sectors Size Id Type

/dev/mmcblk1p1 * 8 42605 42598 20.8M c W95 FAT32 (LBA)

/dev/mmcblk1p2 42608 325475 282868 138.1M 83 Linux
```

We do not have a second partition for our root file-system, but I said that we
where going to deploy a “Dual Copy” strategy
([if you need to refresh your
memory regarding update strategies](https://mkrak.org/2018/01/10/updating-embedded-linux-devices-part1/)).
What will happen is that when we run our first update the update image will run
a script (which is embedded) that will do the eMMC setup. This could have been
taken care of during “build-time” using a custom WIC file but this will work as
well for our test.

Actually lets take a look at the `sw-description` file that is in
[meta-swupdate-boards](https://github.com/sbabic/meta-swupdate-boards):

```bash
software =
{
 version = "0.1.0";
 beaglebone = {
     hardware-compatibility: [ "1.0"];
     stable : {
             copy1 : {
                 images: (
                     {
                         filename = "core-image-full-cmdline-beaglebone.ext4.gz";
                         device = "/dev/mmcblk1p2";
                         type = "raw";
                         installed-directly = true;
                         compressed = true;
                     }
                 );
                 scripts: (
                     {
                         filename = "emmcsetup.lua";
                         type = "lua";
                     }
                 );
                 uboot: (
                     {
                         name = "boot_targets";
                         value = "legacy_mmc1 mmc1 nand0 pxe dhcp";
                     },
                     {
                         name = "bootcmd_legacy_mmc1";
                         value = "setenv mmcdev 1;setenv bootpart 1:2; run mmcboot";
                     }
                 );
             };
             copy2 : {
                 images: (
                     {
                         filename = "core-image-full-cmdline-beaglebone.ext4.gz";
                         device = "/dev/mmcblk1p3";
                         type = "raw";
                         installed-directly = true;
                         compressed = true;
                     }
                 );
                scripts: (
                     {
                         filename = "emmcsetup.lua";
                         type = "lua";
                     }
                 );
                 uboot: (
                     {
                         name = "boot_targets";
                         value = "legacy_mmc1 mmc1 nand0 pxe dhcp";
                     },
                     {
                         name = "bootcmd_legacy_mmc1";
                         value = "setenv mmcdev 1;setenv bootpart 1:3; run mmcboot";
                     }
                 );
            };
        };
    }
}
```

Here we can see that there is a `emmcsetup.lua` script that will take care of
the eMMC setup for us. We can also see an example of how a “Dual Copy” strategy
can be setup using SWUpdate. The boot part switching is done using the `uboot`
handlers that will make changes to U-boot environment variables when performing
an update.

It is now time to do an update, lets browse to the SWUpdate web-interface and
upload the `update-image-beaglebone.swu` file that we got from our Yocto build.

```bash

Waiting for requests...
[network_initializer] : Main thread sleep again !
SWUPDATE successful !
[update_bootloader_env] : Updating bootloader environment
[start_lua_script] : Calling Lua /tmp/emmcsetup.lua
[start_lua_script] : Script output: Post installed script called script end
[install_single_image] : Found installer for stream core-image-full-cmdline-beaglebone.ext4.gz raw
[start_lua_script] : Script output: Checking that no-one is using this disk right now ... FAILED Disk /dev/mmcblk1: 1.8 GiB, 1920991232 bytes, 3751936 sectors Units: sectors of 1 * 512 = 512 bytes Sector size (logical/physical): 512 bytes / 512 bytes I/O size (minimum/optimal): 512 bytes / 512 bytes Disklabel type: dos Disk identifier: 0x07679a7d Old situation: Device Boot Start End Sectors Size Id Type /dev/mmcblk1p1 * 8 42605 42598 20.8M c W95 FAT32 (LBA) /dev/mmcblk1p2 42608 327017 284410 138.9M 83 Linux >>> Script header accepted. >>> Script header accepted. >>> Script header accepted. >>> Script header accepted. >>> Created a new DOS disklabel with disk identifier 0x07679a7d. /dev/mmcblk1p1: Created a new partition 1 of type 'W95 FAT32 (LBA)' and of size 20.8 MiB. /dev/mmcblk1p2: Created a new partition 2 of type 'Linux' and of size 138.9 MiB. /dev/mmcblk1p3: Created a new partition 3 of type 'Linux' and of size 138.9 MiB. /dev/mmcblk1p4: Done. New situation: Disklabel type: dos Disk identifier: 0x07679a7d Device Boot Start End Sectors Size Id Type /dev/mmcblk1p1 * 8 42605 42598 20.8M c W95 FAT32 (LBA) /dev/mmcblk1p2 42608 327017 284410 138.9M 83 Linux /dev/mmcblk1p3 327018 611427 284410 138.9M 83 Linux The partition table has been altered. Calling ioctl() to re-read partition table. The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or kpartx(8). Syncing disks. script end
[start_lua_script] : Calling Lua /tmp/emmcsetup.lua
Installation in progress
[network_initializer] : Valid image found: copying to FLASH
[extract_files] : Found file: filename core-image-full-cmdline-beaglebone.ext4.gz size 31829134 required
[extract_files] : Found file: filename emmcsetup.lua size 1691 required
[check_hw_compatibility] : Hardware compatibility verified
[check_hw_compatibility] : Hardware beaglebone Revision: 1.0
[parse_bootloader] : Bootloader var: boot_targets = legacy_mmc1 mmc1 nand0 pxe dhcp
[parse_bootloader] : Bootloader var: bootcmd_legacy_mmc1 = setenv mmcdev 1;setenv bootpart 1:3; run mmcboot
[parse_scripts] : Found Script: emmcsetup.lua
[parse_images] : Found compressed Image : core-image-full-cmdline-beaglebone.ext4.gz in device : /dev/mmcblk1p3 for handler raw
[parse_hw_compatibility] : Accepted Hw Revision : 1.0
[extract_file_to_tmp] : Found file: filename sw-description size 1138

Software Update started !

File information: update-image-beaglebone.swu size: 31832576 bytes
```

Success! We need to reboot the device for the update to have any effect. There
is a button conveniently placed in the web-interface to trigger a reboot.

We can see now that the devices has changed root file-system part to
`/dev/mmcblk1p3` which was created during our update as can be seen in above
log.

```bash
root@beaglebone:~# mount
/dev/mmcblk1p3 on / type ext4 (rw,relatime,data\=ordered)
< ... >

```

NOTE! SWUpdate is built without “signed image” support by default in Yocto,
there are means to enable it but I will not cover it at this time.

## Conclusion

SWUpdate is a mature and feature rich update framework which is fairly easy to
setup. I have only covered a fragment of what SWUpdate can do but hopefully it
can be helpful to someone.

There is of course a learning curve to using SWUpdate, that is because there
are so many configuration options you need to read-up on what you want to use
and configure appropriately. But this is also the goal of the project, to be a
framework and not an out-of-the box solution.

Also even though there is a fairly good starting point when using Yocto and the
SWUpdate layers to deploy a “Dual Copy” strategy (on a BBB and RPi3 at least),
this is only for testing purposes and is not in anyway production ready and
addition work is needed to make it robust. We are lacking roll-back support is
one example of features that need to be implemented for better robustness when
using a “Dual Copy” strategy.

One thing that is lacking in the project is testing. From what I can see there
are no unit tests nor integration test. To able to maintain such a project
which aims to be flexible you need automation to test the very rich feature
list. I checked the [road-map](https://sbabic.github.io/swupdate/roadmap.html)
of the project and sadly it is not even listed there, instead the focus is
feature on growth.

I am not criticizing as SWUpdate is mainly a community based project meaning
that people develop this on their free time and it is understandable that some
infrastructure is missing. If you have a lot of free time go over to SWUpdate
and start hacking!
