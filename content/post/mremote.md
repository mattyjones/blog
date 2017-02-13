---
date:        "2012-03-30"
title:       "Shell Wrapper for RDesktop"
# description: "Shell script wrapper for rdesktop"
tags:        ["shell programming", "rdesktop", "rdp", "mremote"]
topics:      ["Scripts" ]
slug:        "rdesktop-shell"
---

I use Linux/OSX almost exclusively when it comes to system administration, even if the servers themselves are Windows.  Sometimes this is not the easiest or best thing to do but many times it is much more efficient and quicker to write a script to do what I need to do, and many times those scripts are easier to write in *nix.  It may look like your keyboard threw up on the screen but that one line of sed could save hours over the course of a week.  With that being said, onward.

[MRemote][1] is a good program for accessing multiple servers with Windows RDP.  The tabbed interface and point and click configuration makes logging into and working on several servers at once very easy.  Unfortunately there is no direct port for *nix, there are several comparable programs for KDE, GNOME, and even OSX but they all lack the simplicity and are more cumbersome to make do what you want.  I use a mix of OSX and Linux so anything I write needs to be cross-platform and nothing currently available was, my needs also don't dictate that I need a fancy graphical program either.

I ended up using [rdesktop][2], a commandline tool available as either a source download or through [MacPorts][3].  Most Linux distributions should also have it prebuilt in their respective repositories.  When launching rdesktop you have the ability to set a lot of the typical options as I use this fairly often I wrote a simple script to set all these for me.

{{< highlight text >}}

#! /bin/bash
#
# remote.sh
# This will take a server hostname, compare it against a file, translate it to an IP
# and launch rdesktop with a specific set of opitions. The file can be created using
# the script MRConversion. You can use your own file as well, the format is
# two columns: hostname IP Address. As of now it will also do simple pattern matching
# ex. user enters db and servers are in the form *db*  will open in seperate windows.
#
# Usage:
#
# 1) copy and place this into a text editor and save it as remote.sh
# 2) chmod +x remote.sh
# 3) remote.sh <servername>
#
#
#matt jones
#updated 12/02/11
#copyright GPL v2

declare Input="$1" # the server hostname or pattern you wish to connect to
declare ServerList="/Users/matt/bin/scripts/ServerList.txt" # MRConversion used for the translation
declare ServerName="" # the server name, used to ID the window
declare LocalSharedDisk="$HOME" # set the directory to share with the remote host
declare SharedDiskName="$USER" # the name of the shared disk on the remote system
declare Resolution="1024x768" # the desired resolution, replace -g $Resolution with -f for full screen
declare UserName="" # the desired login username

#---this will give servers matching the criteria---#
set $(grep -c -i $Input $ServerList)
declare ServerCount=$1

# go through $ServerList and match the hostname(s) to IP and start rdesktop
for (( i = 1; i <= $ServerCount; i++ ))

do
  set $(grep -m $i $Input $ServerList | awk -v i="$i" 'NR == i {print $1}') #the host name
  declare ServerName=$1

set $(grep -m $i $Input $ServerList |  awk -v i="$i" 'NR == i {print $2}')
  declare RemoteServer=$1 #set the IP address from the above statement

rdesktop -g $Resolution -r disk:$SharedDiskName=$LocalSharedDisk -u "$UserName" -0 -T $ServerName -N $RemoteServer &

done

{{< /highlight >}}

That took care of launching the remote session but I still needed a way to not have to remember all the IP's of the various servers.  In MRemote there is a setting that allows you to pull the server list from a SQL table and save it to an XML file.  For those with access to this feature you can take the XML file and run it through the next small script to produce a two column list from which to launch remote.sh.  Normally it is not recommended to parse XML with Sed/Awk but for simple, well-formed, static documents(rare) it can be much simpler then using python or perl.


{{< highlight text >}}

#! /bin/bash
#
# MRConversion.sh
# This file takes the output xml file from mremote and strips out the hostname
# and IP and creates a two-column list in a text file.This file is then read by
# remote.sh which matches the hostname to the IP and launches rdesktop with a specific set of options.
# This is written in bash in OSX 10.7 but has been tested in Slackware Linux and works fine.
# The latest development version of this can be found here github.com/mattyjones/remote
#
#
# Usage:
#
# 1) Copy and paste the contents of this into a text editor and save it as MRConversion.sh
# 2) chmod +x MRConversion.sh
# 3) MRConversion.sh
#
#
#matt jones
#updated 11/28/11
#copyright GPL v2

declare temp="temp.txt" #temp file, nothing special
declare ServerList="ServerList.txt" #the name of the file to be created

create_list()
  { #this is if the file dosen't exist at all
  echo "Please enter the full path of the mremote XML file"
  read InputFile

  # use " as a field seperator and print the second and eighteenth column the send it
  # to sed to chop off the extra lines and save it to a temp file
  awk  $temp

  stat -c %y $InputFile >> $temp #get the creation date of input file and append it to temp file

  sed 's/\(.*\)/\L\1/' $temp > $ServerList #convert everything to lowercase for ease of use
  rm $temp #delete the temp file

  }

update_list()
  { #this runs if a file is detected
  echo "Do you want to update the server list? (yes/no)"
  read Ans
  if [ "$Ans" = "yes" ]
    then
      echo "Please enter the full path of the mremote XML file"
      read InputFile
      # use "as a field seperator and print the second and eighteenth column the send it
      # to sed to chop off the extra lines and save it to a temp file
      awk  $temp
      stat -c %y $InputFile >> $temp #get the creation date and append it to the temp file
      set $(sed -n '$p' $temp) #get the last line of the temp file and set it to Date
      declare Date="$1"

      # Now compare the input file date in the temp file to the date of the input file in the
      # current server list.  If the are the same then the temp file is deleted, the they are
      # different then the temp file is set to all lowercase for ease of use and set to the
      # ServerList variable name.

      if grep $Date $ServerList &> /dev/null #redirect all output to the trash
        echo "The server list is already up to date"
        rm $temp

      else sed 's/\(.*\)/\L\1/' $temp > $ServerList #convert everything to lowercase
        rm $temp
      fi

  elif [ "$Ans" = "no" ]
    then echo #add another function to run here

  else update_list #restart the process if anything else is given
  fi
  }

#test to see if the file is already present, if so then check the timestamp
if [ ! -f $ServerList ]
  then
    create_list #create a new file
else
  update_list #create a new file/check the current file

fi

{{< /highlight >}}

The next step would be to have multiple servers open up within the same window but in multiple tabs like they do in MRemote , I rarely have more than one or two servers open at a time so this is not crucial to me but could come in handy if you needed to make changes across the board to a handful of servers of the same type, ie. AD, DNS, etc.  The latest version of these can be found on my [github][4].

[1]: http://www.mremote.org/wiki/
[2]: http://www.rdesktop.org
[3]: http://www.macports.org
[4]: https://github.com/mattyjones