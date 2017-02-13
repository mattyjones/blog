---
date:        "2014-05-07"
title:       "Monitoring Email Storm Mitigation"
# description: "Stop an ongoing monitoring email storm before it gets out of hand"
tags:        ["SEC", perl", "Icinga", "monitoring" ]
topics:      ["Server Administration"]
slug:        "sec-email-storm-mitigation"
---

No one likes to get flooded with email, most times its just annoying, but it can also mask a real problem.  If in those 2000 emails is a single email saying the database was just corrupted, you really don't want to miss it.  There are several tools and ways to control email, but unfortunately most MTA's don't have a way to set a threshold over time variable.  There is no way to say if N emails go out in N minutes stop sending email.

This solution is just one of many and focuses mostly on Nagios and Icinga.  Using [Simple Event Coordinatorr][1] and the following rule set you can stop an email storm before it gets out of control.  Installing SEC is very simple, most distributions already have it in there repos or you can install it from source, it is a single perl script and man page.  SEC has also been known to work on Windows and OSX as well, I am currently running it on Cent6.4 and Slackware 14.1.

To get started download the latest version of the rule set from Github [here][2] then launch sec with the following command

{{< highlight text >}}

sec -conf=email_throttle.conf -input=/path/to/monitoring/log/file

{{< /highlight >}}

You could also use the '-detach' switch to make it run as a daemon.


The rule starts by parsing the icinga.log for all notifications, if greater than 25 get sent out over a sliding 60s period the action takes place.  The initial action is to send an email notification out, create a context, then stop sendmail and wait 30s.  It will then open up a second input stream, in this case maillog, you need to start tail with '-n 0' or else it will trip instantly.

The second rule parses the maillog to make sure email has indeed been stopped, it looks for a dsn, Delivery Status Notification, of any type to be set out, if it sees one then the second action takes place.  At this point monitoring is stopped on the machine, and a second email is sent out (if possible). Another context is created and we sleep for 30s more.  At this point the final rule sees the context it is looking for has been created and if it sees any more mail it will shutdown the server.

I am well aware of how shutting down a server is a BAD idea, but without manual intervention and troubleshooting this is the only way to be absolute.  The third rule could always be removed and the window increased to give the technician time to look at the issue first-hand but if that won't be the case, or access to the machine is difficult this last rule will at least stop the follow of email without question.  You could also modify it to create a dump, log some data, etc as well before it powers off.

{{< highlight text >}}

type=SingleWithThreshold
ptype=RegExp
pattern=dsn=(\d+)\.(\d+)\.(\d+)
context=maillog_tripped
desc=$0
action=shellcmd shutdown -h now 
window=10
thresh=2

type=SingleWithThreshold
ptype=RegExp
pattern=dsn=(\d+)\.(\d+)\.(\d+)
context=icingalog_tripped
desc=$0
action=pipe 'The email threshold was tripped at %t /n Stopping the monitoring services' /bin/mail -s "Email Storm Alert" %e ;  \
       create mail_tripped_context ; \
       shellcmd /sbin/service ido2db stop
       shellcmd /sbin/service icinga stop
       shellcmd /bin/sleep 30
window=10
thresh=2

type=SingleWithThreshold
ptype=RegExp
pattern=NOTIFICATION
desc=$0
action=eval %e {'email_address'} ;\
       pipe 'The email threshold was tripped at %t /n Stopping the sendmil service' /bin/mail -s "Email Storm Alert" %e ;  \
       create icingalog_tripped ; \
       shellcmd /sbin/service sendmail stop ; \
       shellcmd /bin/sleep 30 ; \
       spawn /usr/bin/tail -n 0 -f /var/log/maillog ; \
window=60
thresh=25

{{< /highlight >}}

[1]: http://simple-evcorr.sourceforge.net/
[2]: https://gist.github.com/mattyjones/d68c7768f0e102ac76dd