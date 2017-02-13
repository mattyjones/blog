---
date:        "2013-10-14"
title:       "Using MariaDB with Icinga 1x"
# description: "Setup MariabDB 5.5 with Icinga 1x"
tags:        ["MariaDB", "CentOS", "Icinga", "monitoring"]
topics:      ["Server Administration" ]
slug:        "mariadb-with-icinga"
---

### Installing From Scratch

When you do an install from scratch and following the [icinga documentation][1] for installing IDOUtils the only changes necessary are:

Add the MariaDB repo to yum, **/etc/yum.repos.d/MariaDB.repo**.  In this case I am using Cent6_64bit and Maria 5.5.  For other choices go [here][2].

{{< highlight text >}}

# MariaDB 5.5 CentOS repository list - created 2013-10-14 21:01 UTC
# http://mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/5.5/centos6-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

{{< /highlight >}}

Now install MariaDB

{{< highlight text >}}
yum install MariaDB-server MariaDB-client

{{< /highlight >}}

The rest of the steps are the same for both Icinga and Icinga-Web

### Migrating from MySQL to MariaDB

**Make sure you back up your database by doing a proper dump, as you will need to reinstall the schema's and any data.**

Stop services

{{< highlight text >}}

service mysql stop
service icinga stop
service ido2db stop

{{< /highlight >}}

Uninstall mysql

{{< highlight text >}}

yum list | grep mysql
yum remove xxx

{{< /highlight >}}

You don't need to uninstall everything as much of it will be reinstalled later.  The bare minimum would be mysql-server.  This will also uninstall the following:

* icinga-idoutils-libdbi-mysql
* nagios-plugins

Install MariaDB

{{< highlight text >}}

yum install Mariadb-server

{{< /highlight >}}

Install IDOUtils

{{< highlight text >}}

yum install icinga-idoutils icinga-idoutils-libdbi-mysql

{{< /highlight >}}

Recreate the database according to the [icinga documentation][3].  This will also need to be done for [Icinga-Web][4] as well.

Restart the services

{{< highlight text >}}

service mysql start
service icinga start
service ido2db start

{{< /highlight >}}

You should be all set to log in.

[1]: https://wiki.icinga.org/display/howtos/Setting+up+Icinga+with+IDOUtils+on+RHEL
[2]: https://downloads.mariadb.org/mariadb/repositories/
[3]: https://wiki.icinga.org/display/howtos/Setting+up+Icinga+with+IDOUtils+on+RHEL
[4]: http://docs.icinga.org/latest/en/icinga-web-scratch.html