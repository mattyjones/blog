---
date:        "2012-04-12"
title:       "Administering Windows Servers with FreeBSD"
# description: "Installing and configuring FreeBSD to administer a Windows environment"
tags:        ["BSD", "Windows Server"]
topics:      ["Server Administration"]
slug:        "windows-with-bsd"
---

FreeBSD, the less talked about cousin to Linux is not only a viable alternative to both Windows and Linux but in some ways is far better than either, it really comes down to personal choice and I am slowly choosing FreeBSD for more and more of my needs. Despite this, installing a graphical desktop on FreeBSD still requires a fair amount of work and some measure of skill.  PC-BSD, FreeBSD with KDE and a graphical installer packaged together alleviates much of the work but for those that don't want or need the baggage that comes with KDE, installing FreeBSD and building what you want is really the best option.

I prefer XFCE as my desktop so I will be detailing my migration to FreeBSD and providing the steps I took to build a complete workstation for administering windows from FreeBSD. I don't actually need XFCE, you could use any window manager but that is what I chose to use.   I only require a few simple tools, a web browser, a RDP client, a text editor, a VPN client, and on occasion access to a window's UNC share.  I started with the [9.0][1] release and installed with all the options, using the entire disk. If you are not familar with installing BSD check out this section of the [handbook][2]. I then configured the networking by hand as I hand as I had specific needs but DHCP should suffice for most people. When asked if I wanted to make any changes prior to reboot I said no and let it reboot.

I logged in as root initially to make some changes to the user account I created and install the necessary software packages. Most Linux distributions by default will allow a user to use su and many will allow a user to shut down the system. BSD by default will not allow these things. To enable a user to shut down and reboot the system using the shutdown command they need to be added to the operator group and to allow them to use su they need to be added to the wheel group.

{{< highlight text >}}

pw usermod -G <username> operator, wheel

{{< /highlight >}}

Next I need a desktop. You have two ways of doing this either by compiling from source using ports or by installing pre-built packages. I chose to use the packages to save time and hassle but you could do it either way. If you wish to do it using ports then you will need to update your ports directory to reflect the most current offerings. I use [portsnap][3] for this but there are other ways. Before you do this, make sure /usr/ports is empty by using the following:

{{< highlight text >}}

cd /usr/ports
rm -rf *

{{< /highlight >}}

Next you want to fetch the ports tree and install it. This will only need to be done once, from this point on you can just use the update command.

{{< highlight text >}}

portsnap fetch
portsnap install
portsnap update

{{< /highlight >}}

A nice feature of BSD is that you can combine commands

{{< highlight text >}}

portsnap fetch install
portsnap fetch update

{{< /highlight >}}

The handbook goes into more details about working with [ports][4] but sense I am doing this with packages I am not going to talk about them any more then necessary.

{{< highlight text >}}

pkg_add -r xfce
pkg_add -r xdm
pkg_add -r xorg

{{< /highlight >}}

These will install XFCE and all the dependencies you need. Even though you are installing XFCE and it depends on having a running X Server, you will still need to install it separately and XORG is as good a choice as any.  Xdm is the display manager, you can use kdm or gdm but this just makes sense to me as I am not planning on using either KDE or Gnome.

After that finishes you still need to do a little housekeeping to get this working properly.  These are all the things that Linux will do behind the scenes but you will need to do them manually. You will need to add the next few lines to rc.conf so that they start upon booting.

{{< highlight text >}}

dbus_enable="YES"
hald_enable="YES"

{{< /highlight >}}

You will then need to create ~/.xinitrc with a text editor for each user with the following

{{< highlight text >}}

exec /usr/local/bin/startxfce4

{{< /highlight >}}

This will tell X that when it starts you want to use XFCE, if this line is omitted you will boot into twm, a very minimal desktop but some people may like it if their system has limited resources.  You start XFCE with

{{< highlight text >}}

startx

{{< /highlight >}}

We now have a basic graphical system but we still need to install other software that many take for granted such as web browsers and email clients. I use gmail for web based mail and I have a windows machine running outlook that I remote into for work but thunderbird is a good mail client and it integrates with outlook nicely if needed. I use the following packages: chromium, xchat, rdesktop, openconnect, and vim.

These can all be installed with the pkg_add -r command. If you would prefer, firefox and thunderbird are available as well. I generally use vim/gvim over vi but that is a personal taste, others may want to use gedit, kate, or joe. It depends on your taste, vi is universal and installed everywhere so I just got used to using it. [Rdesktop][5] is a simple windows RDP client. I wrote a script to handle interactions with it in a previous post.  [OpenConnect][6] is an open source client for Cisco's AnyConnect SSL VPN and works well. One piece of note when using openconnect, a stock vpnc script has the ability to overwrite your dns settings in /etc/resolv.conf, you should rewrite the script to not do this but if you are lazy just use

{{< highlight text >}}

chflags schg /etc/resolv.conf

{{< /highlight >}}

to make the file imuttable, this command is the equilvent to chmod +i to remove the flag type

{{< highlight text >}}

chflags noschg /etc/resolv.conf

{{< /highlight >}}

There you have it, a basic Freebsd setup for administering windows machines. There is still more work to be done Part 2 will cover setting up thunderbird for use with outlook, configuring samba to access windows shares, and a few other odds and ends that make my daily life easy and keep windows far away from me.

[1]: http://www.freebsd.org/where.html
[2]: http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/bsdinstall.html
[3]: http://www.freebsd.org/doc/handbook/portsnap.html
[4]: http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-using.html
[5]: http://www.rdesktop.org/
[6]: http://www.infradead.org/openconnect/
