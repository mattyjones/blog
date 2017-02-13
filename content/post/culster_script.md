---
date:        "2014-03-10"
title:       "Icinga Service Cluster Monitor"
#  description: "Send out a single notification if the same service fails across multiple monitored hosts"
tags:        ["python", "JSON", "Icinga", "monitoring"]
topics:      ["Server Administration"]
slug:        "cluster-script"
---


When monitoring large scale environments, email noise can be a real concern, as can having a single point of failure.  When monitoring services such as ntp where the check is the same on many hosts and has the ability to fail across many or all of them at once, notifications can pile up quickly.  Even if this is not a concern, it's simply not necessary to alert on the same failure across all monitored machines.

This script will allow you to send out a single notification that gives the status of the cluster.  A single output listing Critical, Warning, or Unknown.  Using the ntp example, you create a service check to monitor ntp to make sure it stays in sync.  If for some reason an external event causes an issue, for example an ntp server is bad, or maybe there is a bug in the script, using this script will save you from the email storm of every host alerting for the same issue.

Using the Icinga [Rest API][1] you can get a json or xml output that can then be parsed.  I prefer to use JSON and Python, let's begin.

Initial Configuration
---------------------

Create a service check for ntp that will monitor the offset and make sure it stays within allowable bounds.  This service check should check actively or it can be passive if you are using nsca.  They key is that it has notifications disabled, for ease of use I created a service template that I build upon.

{{< highlight text >}}

define service{
    name                     monitoring-cluster-service
    use                      monitoring-service
    notifications_enabled    0
    register                 0
}

{{< /highlight >}}

This will allow me to just assign this template to all services I want to use the cluster check with.

{{< highlight text >}}

define service{
    use                             monitoring-cluster-service
    host_name                       host1
    service_description             NTP Offset
    check_command                   check_ntp!600
}

{{< /highlight >}}

Cluster Configuration
---------------------

Now we need to setup the cluster.  First we create a template for our clusters.

{{< highlight text >}}

define service{
    name                     monitoring-service
    use                      app-service
    notifications_enabled    1
    contact_groups           admins
    register                 0
}

{{< /highlight >}}

Now we define the cluster service.

{{< highlight text >}}

define service{
    use                             monitoring-service
    host_name                       monqaicinga203
    service_description             NRPE Custer
    servicegroups                   clusters
    check_command                   check_cluster!Service - NRPE!5 ;<cmd> <desc> <threshold>
}

{{< /highlight >}}

At this point you have a service check that will show alarms on the dashboard but in the event of a widespread failure will not flood your email inbox, instead the single cluster script will send out a notification of the events.


The Script
----------

The latest can always be found on my Github.

{{< highlight python >}}

import json
import requests
import sys

Service = sys.argv[1]
CritThreshold = sys.argv[2]

Url = ''
r = requests.get(Url)

JsonData = {}
JsonData = r.json()

def main ():

    CritNum = 0
    WarnNum = 0
    UnKnownNum = 0

    for item in JsonData['result']:

    if Service in item.values():
        if item.get('SERVICE_CURRENT_STATE') == '1':
        WarnNum = WarnNum + 1
        elif item.get('SERVICE_CURRENT_STATE') == '2':
        CritNum = CritNum + 1
        elif ite.get('SERVICE_CURRENT_STATE') == '3':
        UnKnownNum = UnKnownNum + 1  

    if CritNum > CritThreshold:
        print WarnNum, 'machines warning', CritNum, 'machines critical', UnKnownNum, 'machines Unknown'
        sys.exit(2)
    else:
        print Service, 'is above critical threshold'
        sys.exit(0)

if __name__ == '__main__':
    main()

{{< /highlight >}}


[1]: https://wiki.icinga.org/display/Dev/Icinga-Web+REST+API
