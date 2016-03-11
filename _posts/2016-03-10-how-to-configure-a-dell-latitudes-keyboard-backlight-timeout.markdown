---
title: How to configure a Dell Latitude's keyboard backlight timeout
layout: post
categories:
    - blog
    - tech
---

This article is about configuring the timeout which is responsible for turning
off the backlight of a Latitude's keyboard after a keyboard, touchpad or track
stick action. By default is it configured at 10 seconds which I wanted to
change. As I had a hard time finding information about the issue, I hope this
article can help others going through less trouble. If you are using Linux and
are not interested in the history, you can skip right to the
[solution](#solution).

A bit of history
----------------

I bought a Dell Latitude E5450 about a year ago. On paper it combined
everything I wanted, a matte screen (I want to see what's on my screen, not my
face), backlit US keyboard (better for programming) a [pointing
stick][pointing-stick] (my fingers seem to be incompatible with touchpads and
it's great for scrolling) and an OK price. The only alternatives would have
been HP, but they don't have a middle button for their pointstick which is
vital for scrolling, and Lenovo, but at the time they only had Thinkpads
without physical trackpoint buttons(horrible, but I guess a lot of people
complained, because now they all have physical buttons again - too late for
me).

I write "on paper", because it has two big problems. I still don't have a
solution for the first one: there is a  ~500ms delay between hitting a key and
using the track stick or its button. Sounds short, but is very noticeable. At
first I thought it's due to the missing driver support of the current Xubuntu
back then as I also had problems with middle button detection and missing palm
check configuration. But as I still had the delay, but not the other problems
after an update to the latest Xubuntu, I contacted the support. After a lot of
back and forth and even swapping the keyboard a few times (the support first
thought it's a hardware issue and then Dell had a hard time getting me a
keyboard without broken keys or broken backlight so I even got a new unit...)
it turns out it's working according to spec. Such a delay might make sense for
touchpads, but even there it's configurable (palm check). It doesn't make sense
for pointing sticks. But all the arguing in the world did not help, it's
working according to spec and as such not a problem for Dell.

Weird configuration values don't have to be a problem, if they can be changed
easily. But where the track stick delay is not configurable at all, the
keyboard backlight timeout is just very hard to change. On my first unit, I
installed Linux right away so as I encountered this timeout and could not
change it easily, I thought it's my fault for using Linux. I found the below
[solution](#solution) but it took a while to get it to work. Like everything
that doesn't work out-of-the-box does under Linux. On my second unit, a year
later and without documentation on what I did, I figured I just keep Windows,
install Linux with dual boot and whenever I have a problem I can just use the
drivers from Dell under Windows. Well, no. Google revealed many hits with
people having the same problem, but all the Windows solutions used
drivers/software that doesn't exist any more. So currently there doesn't seem
to be an easy way to change the timeout under Windows. I guess the below
[solution](#solution) could still be used from a Linux live CD, but how many
Windows users know how to do that?

Solution
--------

One of my Google searches pointed me to libsmbios but it seemed helplessly
outdated. After more searching I came across the [libsmbios git
repository][libsmbios-git-repository] which finally allowed me to change the
timeout. However, it works only under Linux (tested under Xubuntu 15.10).

So lets get started, first you need to get the sources and compile them.

{% highlight bash %}
$ git clone git://linux.dell.com/libsmbios.git
$ cd libsmbios
$ # use a separate directory for the compiled files
$ mkdir _build
$ cd _build
$ # the README file lists a few dependencies, but I needed a few more
$ sudo apt-get install \
    autoconf \
    automake \
    autopoint \
    doxygen \
    gettext \
    libtool \
    libxml2 \
    libxml2-dev
$ # install into some folder that you can simply delete when done
$ ../configure \
    --prefix=/home/user/Desktop/smbios \
    --exec-prefix=/home/user/Desktop/smbios
$ make
$ make install
{% endhighlight %}

Now you can change into the installation folder and set the timeout:

{% highlight bash %}
$ cd /home/user/Desktop/smbios
$ # prints the current timeout and other information
$ sudo ./sbin/smbios-keyboard-ctl --get-status
$ # print the options you have
$ sudo ./sbin/smbios-keyboard-ctl --set-timeout invalid
$ # set the timeout to one hour
$ sudo ./sbin/smbios-keyboard-ctl --set-timeout 1h
{% endhighlight %}

You are done! The value is stored in the BIOS, so it will outlive a new OS
installation and even a new keyboard.

Final notes
-----------

After 5 years, there seems to be a new release of libsmbios from 3 days ago with an
addition regarding the keyboard backlight, but it does not seem to affect this
solution. Maybe there will be the possibility to configure the timeout in the
BIOS at some point, but currently it isn't - at least not for the E5450.

[pointing-stick]:           https://en.wikipedia.org/wiki/Pointing_stick
[libsmbios-git-repository]: http://linux.dell.com/cgi-bin/cgit.cgi/libsmbios.git/
