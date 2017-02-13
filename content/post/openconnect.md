---
date:        "2013-12-14"
title:       "Installing OpenConnect on OSX 10.9 Maverick"
# description: "How to install OpenConnect on Maverick using MacPorts"
tags:        ["osx", "openconnect"]
topics:      ["Server Administration"]
slug:        "openconnect"
---

[OpenConnect][1] is a great alternative to Cisco's AnyConnect client and is completely cross platform and very easy to setup and configure.

1. Install [Macports](http://macports.org)
2. sudo port install openconnect
3. Install [TunTap](http://tuntaposx.sourceforge.net/)
4. Install a [vpnc-script](http://www.infradead.org/openconnect/vpnc-script.html) to ensure proper dns configuration

You can install openconnect from source but the dependency list can be a little scary if you have never done this sort of thing before.  Even if you are comfortable building software packages yourself, I wouldn't do it, it is simply not worth the effort on a virgin 10.9 install.  To use openconnect, open a terminal and type the following command

{{< highlight text >}}

sudo openconnect -u

{{< /highlight >}}

User is the user name assigned to you and server is the server you would connect to with the AnyConnect client.  Your admin should provide this for you but if you have the ability, start AnyConnect and connect with it, then under the statistics tab you will see a server ip, that is what you are looking for.

[1]: http://www.infradead.org/openconnect/