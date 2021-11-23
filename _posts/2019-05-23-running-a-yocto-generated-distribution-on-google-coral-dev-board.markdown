Updated instructions can be found here
[https://github.com/mirzak/meta-coral/wiki/Quick-start-guide](https://github.com/mirzak/meta-coral/wiki/Quick-start-guide)

Last week I got my hands on a Google Coral Dev Board and finally got some time
to play with it.

![](https://lh3.googleusercontent.com/ySyx1CFwvy7dxA6L7XJCOb4jD6H4UK1DYa6jUUm8EFFbC40ce3YXkWNDKPnQLtATwhSDThPi0Dr5LfJHBzYf6VM7y5W7XT3qYrjWIA=w1000-rw)

If you are not familiar with this board please
[read this article](https://www.theverge.com/circuitbreaker/2019/3/6/18253106/google-ai-developer-new-tools-dev-board-usb-accelerator-camera-coral)
and also pay the
[official product site](https://coral.withgoogle.com/products/dev-board) a visit
for more technical details. But in short it is a Linux based System-on-Module
called Edge TPU Module that is mounted on a development board which contains
some additional peripherals. Some key hardware features of the EDGE TP Module:

-   i.MX8MQ SoC with integrated GPU
-   1 GB LPDDR4
-   8 GB eMMC
-   Google Edge TPU coprocessor (used to accelerate ML models)
-   WiFI / Bluetooth

The Coral Dev Board serves as an evaluation board for the Edge TPU Module but
it can also be used as a single-board computer similar to Rasbperry Pi 3 boards.
The Coral Dev Board comes with Linux distribution called Mendel OS which is
Debian derivative. The board did not actually contain the Mendel OS
pre-programmed, and I had to flash the board to get started. But this was
relativity easy as they do provide pre-built OS images and well documented
procedure how to flash the board. You can find the getting started guide here,

- [https://coral.withgoogle.com/docs/dev-board/get-started/](https://coral.withgoogle.com/docs/dev-board/get-started/)

As I spend a lot of time building Linux distributions using
[Yocto/OE-core](https://www.yoctoproject.org/) and Linux distributions on
embedded Linux devices in general are of interest to me, I had to dig deeper
in what Mendel OS was. I asked nicely on Coral website if the tooling to
generate Mendel OS was available publicly, and I received a prompt response
that it was and I got this link:

- [https://coral.googlesource.com/docs/+/refs/heads/master/GettingStarted.md](https://coral.googlesource.com/docs/+/refs/heads/master/GettingStarted.md)

By following the outlined documentation above, I was able to fetch the source
code of all the components for further inspection. There where no real surprises
when I started looking closer and soon realized that that Mendel OS was built
upon fairly standard components coming from the NXP i.MX BSP for the i.MX8 SoC.
With this information I realized that I would be fairly easy to port this and I
set out to create a Yocto BSP layer to be able to build something that boots on
the Coral Dev Board. I only needed to implement three things in this layer to
get it to boot:

- Recipe for U-boot fork
- Recipe for Linux kernel fork
- Created a recipe to deploy a `boot.scr`, and the source for the script I
  copied from Mendel OS

To build an image with Yocto you can use the following steps, Create directory
where you want to store the environment and change the shell to that location:

Initialize repo manifest:

```bash
repo init -u https://github.com/Freescale/fsl-community-bsp-platform -b warrior
```

Fetch layers in manifest:

```bash
repo sync
```

Clone `meta-clang`:

```bash
git clone https://github.com/kraj/meta-clang.git sources/meta-clang -b warrior
```

Clone `meta-coral`:

```bash
git clone https://github.com/mirzak/meta-coral.git sources/meta-coral -b warrior
```

Setup the environment:

```bash
MACHINE=coral-dev DISTRO=fslc-wayland source ./setup-environment build
```

Add the `meta-clang` layer to bblayers.conf:

```bash
echo 'BBLAYERS += "${BSPDIR}/sources/meta-clang"' >> conf/bblayers.conf
```

Add the `meta-coral` layer to bblayers.conf:

```bash
echo 'BBLAYERS += "${BSPDIR}/sources/meta-coral"'>> conf/bblayers.conf
```

Start baking:

```bash
bitbake core-image-base
```

After the build has finished you should now have a WIC image:

- `tmp/deploy/images/coral-dev/core-image-base-coral-dev.wic.gz`

This is an image that you can write either to the eMMC or an SD card to get a
booting system. To simplify things I changed boot mode of the Coral Dev Board
to boot from SD card instead of eMMC which is the default, and to do this I
adjusted the boot switches to the following,

![](https://coral.withgoogle.com/static/docs/images/devboard/devboard-bootmode-sdcard.jpg)

Since we already built our image, we can now write it to a SD card:

```bash
zcat tmp/deploy/images/coral-dev/core-image-base-coral-dev.wic.gz | sudo dd of=<device> bs=4M
```

Insert the SD card and boot the system, we should now end up at login prompt of a Yocto build distribution:

````bash
FSLC Wayland 2.7 coral-dev ttymxc0

coral-dev login
```

I have not yet done much testing of the different peripherals, but the basic
components (HDMI, Ethernet, USB) should work as I am using the same Linux kernel
as Mendel OS. I know of a couple of things that need some more work as it
depends on user-space components, and here is a very short list:

- WiFi and Bluetooth (need to install correct firmware files)
- Google Edge TPU coprocessor (would be fun to get the ML demo from Mendel OS running in Yocto)

The source code of the Yocto/OE-core BSP layer for Google Coral Dev can be found here

- [https://github.com/mirzak/meta-coral](https://github.com/mirzak/meta-coral)

Happy testing and feel free to reach out if you are interested in getting involved.
