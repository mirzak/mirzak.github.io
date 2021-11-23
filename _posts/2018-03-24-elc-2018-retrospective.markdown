My last post was written from Portland, Oregon where I was attending and
speaking at the Embedded Linux Conference. It was definitely a busy week that I
spent there trying to absorb as much as possible from all the sessions and the
“hallway track”, while also connecting with people over beers
(Portland = beer town).

My intention was to write-up a summary of the event from my perspective and here
it is.

This years ELC event was hosted in the downtown
[Hilton Hotel in Portland](http://www3.hilton.com/en/hotels/oregon/hilton-portland-downtown-PDXPHHH/index.html).
The date of event was March 12 – 14 and that was Monday – Wednesday.

## Venue

One thing that surprised me when we entered the Hilton Hotel on Monday morning
was the lack of information/signs in the lobby. I thought first that we arrived
at the wrong Hilton Hotel (do not know actually know if there are multiple
Hilton hotels Portland). I was mostly comparing this experience with past ELCE
events where you are usually greeted at the door and it is very hard to miss
that there is a ELC event going on here. But here there was nothing. We walked
past the huge lobby and reached a couple of stairs, and it was first here we
saw a sign saying that ELC event registration is upstairs. Which brings up the
second problem and last problem/complaint I had with the venue. This particular
venue and event was distributed across several floors and the floors where
quite small, mostly meaning that there where no large open areas. This layout
made the event feel a lot smaller then it actually was because you had people
distributed across several floors and this is how it was most of the day. During
the evening events you would get most the people in one place but then it was
very crowded and loud.

There was one thing that was new at the registration, at least I have not seen
this on earlier events. And that was that you where able to choose the design
of your conference t-shirt. Though they called it “create a custom design” you
where only able to choose from two different templates with no additional
customization, which means that you where able to choose between two different
t-shirts. Note that there there two design for ELC and two for IoT Summit. But
they did actually print the t-shirt design that you choose on site. To me this
was mostly a inconvenience and it did not add much value to the experience. You
choose which design you wanted at registration, waited a day to get a
notification that it was ready to be picked up, waited in line to pick it up
and that was it. I would rather have just picked it up when I registered which
was on Sunday during the pre-registration to avoid the initial lines.

## Overall impressions

Every ELC(E) event that I have been to, there has been something that stands
out. I have mostly been to ELCE, but one year it was all Yocto, the next one
was all Yocto again, 2016 was big focus on software updates and so on.

This year, for me it was RISC-V, the new kid on the block. I have not really
heard about SiFive until they announced their
[HiFive Unleashed](https://www.sifive.com/products/hifive-unleashed/) board,
The First Linux-ready RISC-V Chip. They first announced it in February 2018 at
[FOSDEM](https://fosdem.org/2018/). I had heard of RISC-V but never really
given it much thought or looked closer what it actually is.

RISC-V is an open-source full-featured ISA with origins in UC Berkeley and is
now days standardized and managed by the non-profit organization
[RISC-V Foundation](https://riscv.org/). The members list is impressive with
companies like Google, NVIDIA, Qualcomm, Samsung and many more. Having an open
and free ISA means that there are no license fees nor restrictions on the
entities that implement it, which of course is interesting to the major silicon
vendors in the mobile and embedded market which is exclusively ARM which is
associated with license fees. A free ISA opens some new exciting doors and it
will be interesting to follow what lays ahead.

SiFive hosted a Hackathon along side ELC which I attended. You can read more
about the actual Hackathon [here](https://www.sifive.com/blog/2018/03/03/all-aboard-part-11-risc-v-hackathon-presented-by-sifive/?utm_campaign=Newsletter&utm_source=hs_email&utm_medium=email&_hsenc=p2ANqtz-9UVbS-YikeZ2CEM1f1hcRioEsM4TSfI4-Tg_NMJ7zDUrH3XagUloJlgw2ChJKaSAQWWFOR).

My intention with attending was not be part of the competitions, because I
knew that I could not spend the time necessary. I did have one goal though,
booting something built with Yocto/OE-core on the board, using
[meta-riscv](https://github.com/riscv/meta-riscv) which at the time only had
support for a qemuriscv64 machine. I did not accomplish this completely during
the event but I did manage to do some work which I cleaned up when I got home
and which I
[published, and it got accepted in the upstream meta-riscv layer](https://github.com/riscv/meta-riscv/pull/3).
There is actually still some tests pending to be run on my work as very few
people have the actual board but hopefully it is only minor tweaks to get it
working fully.

![](/assets/images/2018-elc-hackaton.jpg)
*Hacking on the HiFive Unleashed @ SiFive Hackethon*

## Day 1 (Sessions)

On day one I arrived early (do not want to miss anything) and ate of the
complementary breakfast. And then it was off to the Grand Ballroom, venue for
the keynotes. I obviously arrived a bit earlier to get a good seat, which I was
happy about because the room filled up fast.

![](/assets/images/2018-elc-grand-ballroom.jpg)
*Grand Ballroom*

It was hard to capture the full room from that angle but it should give you an
idea of how it was. Note that most of the pictures where taken with my phone
which does not have the best camera.

I sat through all the keynotes which where:

- [Keynote: Welcome – Tim Bird, ELC Co-Chair & Philip DesAutels, IoT Co-Chair](https://www.youtube.com/watch?v=wirx1SwMlbA)

It is always pleasant to listen to Tim Bird and you can feel the passion that
he has for this community and the events that we all like to attend to much.

- [Keynote: Intelligent Internet of Things: Start Analyzing Your Global Device
  Data for Real-Time – Antony Passemard, Product Management Lead – Cloud IoT, Google](https://www.youtube.com/watch?v=Jux_FAzpP30)

In this talk Antony talked about how he feels that “IoT will disappear” and how
it is not interesting to bring up numbers of connected devices but instead talk
about the value that these devices bring by utilizing cloud technologies
combined with machine learning and AI.

The three challenges with IoT that Antony focused on was:

-   Connecting devices securely
-   Scaling (big-data)
-   Actionable insights (ML)

And presented solutions on Google solve these within the Google Cloud IoT
solution.

Overall a very good presentation and I can recommend you watching the full
video which I linked.

- [Keynote: Designing the Next Billion Chips: How RISC-V is Revolutionizing
  Hardware – Yunsup Lee, Co-Founder and CTO, SiFive](https://www.youtube.com/watch?v=pxd93jb1OAk)

Yunsup Lee from SiFive explained on how they are planing on changing the
hardware market (silicon) taking inspiration from software, that is basing it
on open-source and RISC-V. In the hopes to reduce cost and lead-times making
custom chips available to the masses.

One interesting analogy from the talk is that the effort to order a custom SoC
should be as easy as ordering pizza.

Watch the full presentation!

After the keynotes it was time to head up to the 23rd floor to the SiFive
Hackethon for an introduction. I did not stay for the full introduction as I
wanted to attend the Mender BoF which was at the same time.

- [BoF: Mender, Current and Future Status of the Open Source Project – Eystein Stenberg, Mender.io](https://www.youtube.com/watch?v=SAWxFqzH-dY)

This obviously interested me as I am involved on the community side of the
[Mender project](https://mender.io/), which I have been part for the last two
years and it is always nice to talk with people that have used the project and
discuss the future road-map.

Eystein Stenberg gave an introduction with an demo of the Mender server which
is used for device and deployment management.

The current focus for the Mender development team is:

- Automatic U-boot patching
    - This is basically a bash script that is able to patch most of the upstream
      boards in U-boot with the required   integration points for Mender. Mender
      uses U-boot to do the A/B rootfs switching and for roll-back.

- Mender community integrations
    - Index of community ports of Mender

- Free CI
    - Mender has extensive CI environment that they plan on making available to
      the community. TBD how this will work.

Then there was a free discussion. Here are some keywords from my notes:

- Schedule updates
    - cron like
    - Salting out / campaign management
- Factory reset?
    - Overlay FS?
- Chain devices?
    - http proxy?
- Delta updates?
    - Payed alternative CI service
- Dynamic groups

After the Mender BoF it was back to the 23rd floor for the SiFive Hackethon,
and I stayed here for a while working until late afternoon when I joined the
following session.

- [OpenEmbedded/Yocto on RISC-V – New Kid on the Block – Khem Raj, Comcast](https://www.youtube.com/watch?v=TdsmjqWJmfc)

Obviously I went to this as I was attending the Hackethon and working on
getting Yocto/OE-core to boot on the HiFive Unleashed board.

In this talk Khem Raj provides an good overview of the current state of the
RISC-V port (linux, gcc, qemu and more) in general and what the status is of
meta-riscv and RISC-V in OE-core.

To summarize (what works):

- You can build
    - core-image-minimal
    - core-image-base
    - core-image-fullcmdline
- You can generate SDK/eSDK
- You can build and boot RISC-V qemu

To summarize (needs work!)

- Graphical target images (core-image-sato/x11/weston)
- QEMU self-test
    - Important to get working, large run-time test suite provided by Yocto/OE-core.

I approached Khem after his talk and told him that I was working on the HiFive
Unleashed board support for meta-riscv. He later joined us at the 23rd floor
and helped me out with some issues I was having. Big thanks for that!

- [Update My Board! – Mirza Krak, Endian Technologies AB](https://www.youtube.com/watch?v=ULouYBeoNBY&t=1s)

The day of my presentation on how to integrate an open-source software update
solution to a custom board. My time-slot was 10:50am and I decided that I would
skip the keynotes this morning and found an empty room to test out all the
technical stuff, that is connecting my laptop to a projector and to actually
see that it works smoothly. I had tested back home but was a bit paranoid and
wanted to make sure that it all still worked, and it did!

Rest of the time I spent walking around being nervous. This was the first time
for me speaking to a larger audience (20+) and in my field ELC is the biggest
conference, so the pressure was on. I actually went to the last keynote which
was in the same room where my session was planned (Grand Ballroom II) to be
nearby and to be able to setup my presentation straight after the keynotes
finished.

Overall I am happy with how the session went and there was good number of people
attending. I am happy that there where many questions at the end of my session
as it always more fun to interact with the audience. Though I did the rookie
mistake and did not repeat the questions on the microphone and it can be hard
to hear the questions on the recording. It is an learning journey and hopefully
if I am given the opportunity to speak again I will remember this lesson.

When my session was over it felt a bit lighter to walk around at the event and
I mostly enjoyed the hallway track for the rest of day and hacking on the 23rd
floor.

There was a sponsor/technical showcase at the end of the day, as there normally
is at these events, but it was pretty much the same what was shown during the
normal sponsor showcase which was available during the day + beer :).

Last day of the conference was packed with content as well. I decided to attend
the keynotes in the morning. The one that stuck with me was:

- [Keynote: Calm Technology: Design for the Next 50 Billion Things – Amber Case, Author and Fellow at Harvard’s Berkman Klein Center](https://youtu.be/wKHa889Q5Uw)

In this talk Amber Case talks about how we are all cyborgs and how to apply
Calm Technology in our design of technology. Best experienced if watched!

After the keynotes I again went to the 23rd floor where it was time to finish
up the SiFive Hackaton. There was a demo session where attendees presented what
they have accomplished during the event.

Here are the winners:

The winner of the JavaScript benchmark competition, ported
[SpiderMonkey](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Releases/1.8.5)
to RISC-V and ran the
[SunSpider](https://webkit.org/perf/sunspider/sunspider.html) JavaScript
benchmark on the HiFive Unleashed board. I do not actually have results of the
benchmark but still an impressive achievement in a short time period.

The winner of the coolest demo was the was a team running OpenEmbedded on the
HiFive Unleashed board and running python.

I also presented the patches that I was preparing for meta-riscv which are now
upstream:

After the Hackethon finished up I was mostly attending the “hallway track”
until this session:

- [Living on Master: Using Yocto Project, Jenkins and LAVA for a Rolling Release – Tim Orling, Intel](https://youtu.be/l6NwYGbWO5s)

Tim Orling presents how they do rolling releases using Yocto, Jenkins and LAVA
at the Intel Open Source Technology Center following the “Linux kernel model”.

Some really great insights and one interesting part is that Intel is planing to
open-source their whole infrastructure so that more can use it and improve it.
Can not wait to see this and hopefully set it up my self.

This was the last session that I attended at the ELC 2018 event and pretty much
called the conference here and had to “move” to another location as I was
staying with a group of people that where leaving on Wednesday but I stayed
until Thursday.

I did join the SiFive team and Hackathon attendees at
[Henry\`s](https://henrystavern.com/menu.php?c=portland) in the evening and we
had a blast and was a really nice way of finishing of my trip.
