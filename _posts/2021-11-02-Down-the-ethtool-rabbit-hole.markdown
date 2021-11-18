---
layout: post
title:  "Work in progress"
published: false
---

This is a short story of one of those problems where you scratch your head, but
curiosity does not allow you to leave questions unanswered so you dig deeper
and deeper until the revelation comes. In this case it was also an necessity
and not just curiosity.

I was working on Raspberry Pi Compute Module 4 (CM4) based hardware, and this
story is specifically about the Ethernet interface. CM4 supports up to 1000Mb
Ethernet speeds but due to a hardware issue we could not support the maximum
speeds on our carrier board.

My simple task (I thought) was to make sure that the Ethernet interface only
advertises supported link modes. This is related to Ethernet
[autonegotiation](https://en.wikipedia.org/wiki/Autonegotiation){:target="_blank"}
and can be summed up as:

> During autonegotiation, each device declares its technology abilities, that
> is, its possible modes of operation. The best common mode is chosen, with
> higher speed preferred over lower, and full duplex preferred over half duplex
> at the same speed.

The default supported and advertised modes are:

>```bash
>$ ethtool eth0
>Settings for eth0:
>    Supported ports: [ TP MII ]
>    Supported link modes:   10baseT/Half 10baseT/Full
>                            100baseT/Half 100baseT/Full
>                            1000baseT/Half 1000baseT/Full
>    Supported pause frame use: Symmetric Receive-only
>    Supports auto-negotiation: Yes
>    Supported FEC modes: Not reported
>    Advertised link modes:  10baseT/Half 10baseT/Full
>                            100baseT/Half 100baseT/Full
>                            1000baseT/Half 1000baseT/Full
>    Advertised pause frame use: Symmetric Receive-only
>    ...
>```

We can easily manipulate advertised link modes using
[ethtool advertise](https://man7.org/linux/man-pages/man8/ethtool.8.html){:target="_blank"}:

> ```bash
>$ sudo ethtool -s eth0 advertise 0x00F
> ```

`0x0F` above means only advertise modes up to 100baseT Full
(see [ethtool man page for more info](https://man7.org/linux/man-pages/man8/ethtool.8.html){:target="_blank"}:).

Inspecting the configuration we get:

>```bash
>$ ethtool eth0
>Settings for eth0:
>    Supported ports: [ TP MII ]
>    Supported link modes:   10baseT/Half 10baseT/Full
>                            100baseT/Half 100baseT/Full
>                            1000baseT/Half 1000baseT/Full
>    Supported pause frame use: Symmetric Receive-only
>    Supports auto-negotiation: Yes
>    Supported FEC modes: Not reported
>    Advertised link modes:  10baseT/Half 10baseT/Full
>                            100baseT/Half 100baseT/Full
>    ....
>```

So far everything works as expected, and we can see above that we no longer
advertise the 1000baseT modes.

After initial testing with `ethtool`, I started looking in to a more permanent
solution, because this needs to be applied on boot and we had a larger fleet of
devices to apply this change to.

We are using a custom Linux distribution based on OE-core/Yocto Project and we
use
[systemd-network](https://www.freedesktop.org/software/systemd/man/systemd.network.html){:target="_blank"}
as a network manager. There is a framework to apply network device configuration
in systemd-network called [systemd.link](https://www.freedesktop.org/software/systemd/man/systemd.link.html){:target="_blank"}
which is exactly what we need right now.

The change is simple really (I thought), we just need to create a `.link` file
that matches our network interface with the correct configuration options.

**Example: /etc/systemd/network/10-internet.link**

```bash
 [Match]
 OriginalName=eth0

 [Link]
 Advertise=10baset-half
 Advertise=10baset-full
 Advertise=100baset-half
 Advertise=100baset-full
```

This would be the equivalent configuration to the `ethtool` commands we ran
earlier.

I was very happy with these changes, until I tested them:

```bash
systemd-udevd[175]: ethtool: Cannot get device settings for eth0 : Invalid argument
systemd-udevd[175]: Could not set advertise mode: Invalid argument
```

What?! Probably my fault I thought, and I went back to reading the documentation
for [systemd.link](https://www.freedesktop.org/software/systemd/man/systemd.link.html){:target="_blank"}
to check if I had missed something. I also tried to experiment with other
configuration options in my `.link` file, but whatever I put in there I always
got the same error message.

Still thinking that my understanding of systemd.link was not correct, I tried
a different approach. I created a simple udev rule:

```bash
ACTION=="add", SUBSYSTEM=="net", KERNEL=="eth*", RUN+="/usr/bin/ethtool -s $name advertise 0x00F"
```

Above did not give me a error message (or it was hidden somewhere by udevd),
but I could see that the configuration was not changed from the defaults with
`ethtool`.

This at least confirmed that my systemd.link file was probably OK, so I went
back to debug this. I only had the error messages from systemd-udevd to go on,
so I went to the systemd source code to look them up.

I focused on the first message:

```bash
systemd-udevd[175]: ethtool: Cannot get device settings for eth0 : Invalid argument
```

and found it in [src/shared/ethtool-util.c](https://github.com/systemd/systemd/blob/f533135c6ce8de641ebf9cdf8deb53faa723479f/src/shared/ethtool-util.c#L974)


```C
r = get_glinksettings(*fd, &ifr, &u);
if (r < 0) {
        r = get_gset(*fd, &ifr, &u);
        if (r < 0)
                return log_debug_errno(r, "ethtool: Cannot get device settings for %s : %m", ifname);
}
```

Not sure which of these functions where failing but both of them rely on calling
`ioctl(SIOCETHTOOL)` and this was where the `EINVAL` was originating from. This
meant it was time to jump over to the Linux kernel code instead.

CM4 uses the
[bcmgenet Ethernet driver](https://github.com/torvalds/linux/blob/master/drivers/net/ethernet/broadcom/genet/bcmgenet.c),
and I assumed that this code was involved to propagate link settings.

Indeed, there is a `ethtool_ops` struct that defines ethtool supported functions.
The ones that I was looking for are below since we got an error message when
trying to get link settings:

```C
/* standard ethtool support functions. */
static const struct ethtool_ops bcmgenet_ethtool_ops = {
    ...
    .get_link_ksettings = bcmgenet_get_link_ksettings,
    .set_link_ksettings = bcmgenet_set_link_ksettings,
    ...
};
```

And looking at `bcmgenet_get_link_ksettings`, we see the reason of the `EINVAL`:

```C
static int bcmgenet_get_link_ksettings(struct net_device *dev,
                       struct ethtool_link_ksettings *cmd)
{
    if (!netif_running(dev))
        return -EINVAL;

    if (!dev->phydev)
        return -ENODEV;

    phy_ethtool_ksettings_get(dev->phydev, cmd);

    return 0;
}
```

The `bcmgenet` driver requires the Ethernet interface to be up and running
before it will accept any ethtool commands, the same guard exists on all
ethtool_ops.

Looking at the source code for other Ethernet drivers, the behavior of
`bcmgenet` seems to be an exception rather then the rule. This probably relates
to some hardware limitation where Media Access Control (MAC) and PHY
(transceiver) are tightly coupled.

CM4 uses the BCM54210 PHY, but as I could not find a proper datasheet for this
component I stopped digging here.

## Conclusion

I think this post demonstrates the beauty of open source and the ability to
dig trough large software stacks to troubleshoot tricky problems.

It also demonstrates some of the usability challenges we are faced in the Linux
world. I make my living solving problems like these (keep them coming), but
an average Linux user would probably struggle and lets remind our self that
user facing error was:

```bash
systemd-udevd[175]: ethtool: Cannot get device settings for eth0 : Invalid argument
```

But an average Linux user probably would not mess with advertised link modes of
the Eternet PHY, so I guess it is all good in the end.

Hopefully it was an enjoyable read at least.

Some might wonder how I solved this in the end, using a systemd service:

```bash
[Unit]
Description=Ethtool configuration to limit to 100Mb speed for the CM4 MAC/PHY
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/sbin/ethtool -s eth0 advertise 0x00F
Type=oneshot

[Install]
WantedBy=multi-user.target

```

If you know of a better way, please let me know.
