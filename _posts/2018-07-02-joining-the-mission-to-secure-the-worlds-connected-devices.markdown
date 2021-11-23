Exciting times!

I have gotten the opportunity to start working for
[northern.tech](https://northern.tech/), the commercial entity behind the
[Mender](https://mender.io/) project. Before I start writing about my
relationship with Mender I wanted just link to
[Northern.tech culture blog](https://northern.tech/blog/tag:culture), to give
you an idea of the company culture which is always a deciding factor for me when
choosing an employer and there are some interesting ideas in the posts that
might inspire you and your company.

I have been a community member of the Mender project since 2016, doing various
things. It started out with me adding support for the popular Raspberry Pi
boards. The initial reference boards where Beaglebone Black and qemuarm, I did
not have a BBB board at the time but I wanted to try Mender out, so of course
you port it to what you have laying around on the desk. Now days the the
Raspberry Pi 3 is one of the official supported boards and the entry point for
many to the project.

Another contribution worth mentioning that I did during my time as a community
member, was to add support for updating UBI volumes on raw NAND based devices.
Initially Mender only supported block based devices, that is eMMC/SD. Should be
noted that I only did the initial UBI work, this was later picked up by the
project and integrated in QA/CI and became officially supported.

I have enjoyed following the projects from it early days and been part of the
community as it has grown and I look forward to continue being part of it now
as an employee.

Going from a community member to employee, it makes sense to focus on tasks
that will enable the community even further and one of the first things that I
am looking at and that we have planned is to a take look how we can improve
community integration of boards. The goal with this is:

- Better overview of board support
    - What boards are supported?
    - What is the status of the integration
- Easier to get started on one of the supported boards
- Easier to contribute support for your board

You can follow this work here:

-   [Community integrated boards](https://tracker.mender.io/browse/MEN-1962)

Feedback is always welcome, so please join the discussion. I really hope that we
can collaborate on this.

I would also like to extend a thank you to my previous employer
[Endian Technologies AB](https://endian.se/). I have really enjoyed working
here and we part ways on the best terms I believe. I have always felt the
support of the company and of my colleagues for all my idea and endeavors. If
you are looking for a job in Gothenburg, Sweden, they now have a slot to fill :).

Looking ahead of what will happen regarding posts on this site, I will probably
pause the article series that I have started on software updates solutions,
since now I am getting paid working a specific project. I do believe that I
could continue this without being effected by that, but I also want to write
about other things. I have not decided what yet. It will probably be more Yocto
focused posts and probably more stuff about hardware.

Stay tuned!

NOTE!

For more technical insights about Mender take a look at one of my previous posts:  
– [Updating Embedded Linux Devices: Mender](https://mkrak.org/2018/02/09/updating-embedded-linux-devices-mender/).
