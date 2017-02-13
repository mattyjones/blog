---
date:        "2013-08-12"
title:       "Arch Linux Cheatsheet"
# description: "Common commands and one-liners for setting up and working with Arch Linux"
tags:        [ "Arch Linux" ]
topics:      [ "Server Administration" ]
slug:        "arch-cheatsheet"
---

When it comes to administering servers, even Windows servers, Linux is a much better choice in my opinion.  Many of the tools needed come pre-installed and are generally of a higher quality then those developed for Windows.  It may seem counter intuitive and it really depends on what you do on a day to day basis but for those that have a need or a desire, this cheat sheet may help you out.  It is geared around Arch Linux, my preferred distribution, but many of the commands will work with most if not all versions of Linux.

I am not going to go through how to install Arch as the [documentation][1] for this is already excellent and many people keep it as up to date as possible.  This is merely a partial list of common commands that I find myself using often enough that I wrote them down.  I often keep this list on a flash drive so that when I am standing up new servers I can cut and paste necessary sections rather than have to type them in constantly.

The list is broken up into sections to keep things somewhat neat and to make it easy for me to update it as necessary.  Feel free to make any suggestions for additions to this list.

### PACKAGE MANAGEMENT

Pacman is the default tool for installing, updating and otherwise managing your Arch Linux installation.  The full documentation can be found [here][2] but below are the most common commands.

{{< highlight text >}}

pacman -S    # install a package
pacman -Syu  # upgrade the system
pacman -Rn   # remove a package with and backup configuration files
pacman -Rsn  # remove a package and all dependencies

{{< /highlight >}}

### Commonly Installed Packages

These are packages that I use on my workstation.  You may not need all of these or you may need more.  On a high level I use [Fluxbox][3] as my desktop and [QEMU][4] for all my virtual machines.  A few of the less commonly known programs in the list are [slim][5](graphical login manager), [tilda][6](used for overlaying a terminal window on the background), and [vifm][7](file manager with vi bindings).

**X Packages**

* xorg-server
* xorg-apps
* xorg-message
* xf86-video-noveau
* xlockmore
* xorg-xcalc


**Development Packages**

* base-devel
* tk
* python2
* git
* vim
* texlive-most
* lua

**GUI Packages**

* rxvt
* slim
* fluxbox
* chromium
* feh
* conky
* tilda
* vifm
* xpdf

**Virtualization Packages**

* qemu

**System Packages**

* ntpd
* openssh
* rdesktop  
* unzip
* sudo

**Office Packages**

* libreoffice
* libreoffice-en-US

### Networking

Networking has always been a tricky thing to configure. Sometimes it works right out of the box, other times it takes a little tweaking. I have had very good luck with the [documentation][8] and these commands are generally all that is necessary for the most common setups.

**Using DHCP**

{{< highlight text >}}

lspci -v | more             # see what devices are present on the system
dmesg | grep                # check to see if the module was loaded
ls /sys/class/net           # get the name assigned to the device
ip link set up              # enable the device
ip link show                # check the status of the device
dhcpcd                      # enable dhcp for the device
ip addr show                # check the ip address of the device
systemctl enable dhcpcd     # enable dhcp at startup for the device

{{< /highlight >}}
**Static IP**

Place this into /etc/systemd/system/network.service

{{< highlight text >}}

[Unit]
Description=Wired static IP setup
Wants=network.target
Before=network.target
BindsTo=sys-subsystem-net-devices-.device
After=sys-subsystem-net-devices-.device

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/ip link set dev  up
ExecStart=/usr/bin/ip addr add  dev
ExecStart=/usr/bin/ip route add default via

[Install]
WantedBy=multi-user.target
systemctl start network
systemctl enable network

{{< /highlight >}}

To set the DNS

{{< highlight text >}}

echo "nameserver xxx.xxx.xxx.xxx" >> /etc/resolv.conf

{{< /highlight >}}


### X Windows Configuration

This is a highly subjective section and is tailored towards my needs and wants. Everybody is different but this should give you an idea of where to start or hopefully save you the time of reading through numerous wiki pages to figure out one thing.

**Edit the bash profile to use a graphical login manager**

{{< highlight text >}}

cp /etc/skel/.bash_profile ~/                            # default bash profile
echo "[[ -z $DISPLAY && $XDG_VTNR -eq 1]]" exec slim     # to start slim on startup

{{< /highlight >}}

**Tell slim to start fluxbox upon successful login**

{{< highlight text >}}

touch ~/.xinitrc
echo "#!/bin/sh" >> ~/.xinitrc
echo "exec startfluxbox" >> ~/.xinitrc

{{< /highlight >}}

**Enable slim**

{{< highlight text >}}

systemctl enable slim.service

{{< /highlight >}}

**Set the background**

{{< highlight text >}}

fehbg --bg-scale ~./fluxbox/backgrounds/xxx.jpg/png/etc
chmod 770 ~/.fehbg

{{< /highlight >}}

**Misc stuff**

{{< highlight text >}}

usermod -aG log                       # this will allow the regular user access to the system log files
cp /etc/conky/conky.conf ~/.conkyrc   # default conky script

{{< /highlight >}}

**Fluxbox Configuration**

This is again going to depend on what you use and if you even use Fluxbox vs K, Gnome, XFCE,etc. These commands should be added to ~/.fluxbox/startup.

{{< highlight text >}}

tilda &                                    # autostart tilda
conky &                                    # autostart conky
xrandr --output DVI-I-1 --right-of VGA-1   # dual monitor (change screens as needed)
sh ~/.fehbg &                              # set the background image

{{< /highlight >}}

### Tilda Configuration

This will fill the remainder of the space not used by conky on my left monitor. Under the *General* tab uncheck *Always on top*. Under the *Appearance* tab check *Enable Transparency* and set *Level of Transparency* to **100%**.  I tend to use dark backgrounds so under *Colors* I use **Green on Black** and I disable the scrollbar and set the following values under *Appearance*.

{{< highlight text >}}

height 75%
width 40%
x-position 300

{{< /highlight >}}

### USB DEVICES

{{< highlight text >}}

dmesg | grep -E "sd[a-z]"         # list all storage devices the system is aware of
mount -t auto /dev/sd? /mnt/usb   # mount the device
umount /dev/sd?                   # unmount the device

{{< /highlight >}}

### KVM/QEMU

I tend to use QEMU with KVM for managing virtual machines due to unreliability I have experienced with virtual box in general. Most people will never have issues with virtual box and it can be easier to setup configurations using it but I found QEMU to be better and coupled with KVM to be faster and more stable. When enabling usb passthrough the device must be unmounted on the host system.

{{< highlight text >}}

gpasswd -a  kvm                                                               # add user to the kvm group
dd if=/dev/cdrom of=mycdimg.iso                                               # dump the contents of the dvd to the hd for install
qemu-img create -f qcow2 myimage.qcow2 10G                                    # create a dynamic image(qcow2)
qemu-img create -f raw myimage.qcow2 10G                                      # create a static image(qcow2)
qemu-system- -enable-kvm -m 1024 -cdrom xxx.iso -boot d -name xxx xxx.qcow2   # nstall the guest OS
qemu-system- -enable-kvm -m 1024 xxx.qcow2                                    # this will run the guest OS
qemu-system- -m 512 -usbdevice host:xxx:xxx xxx.qcow2                         # enable usb pass through
qemu-img convert -O vdi xxx.qcow2 xxx.vdi                                     # convert the image for use with vbox

{{< /highlight >}}

### MISC Commands

{{< highlight text >}}

ntpd -sd                          # sync the time
systemctl start ntpd              # start ntpd
systemctl enable ntpd             # start ntpd at boot
systemctl start sshd              # start ssh daemon
systemctl enable sshd.service     # start ssh at boot

useradd -m -g [initial_group] -G [additional_groups] -s [login_shell] [username]   # add a normal user account
passwd [username] [password]                                                       # change the password of a normal account

{{< /highlight >}}

[1]: https://wiki.archlinux.org/index.php/Installation_Guide
[2]: https://wiki.archlinux.org/index.php/Pacman
[3]: https://wiki.archlinux.org/index.php/Fluxbox
[4]: https://wiki.archlinux.org/index.php/QEMU
[5]: https://wiki.archlinux.org/index.php/SLiM
[6]: https://wiki.archlinux.org/index.php/Tilda
[7]: https://wiki.archlinux.org/index.php/Vifm
[8]: https://wiki.archlinux.org/index.php/Network_Configuration
