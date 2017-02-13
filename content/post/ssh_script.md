---
date:        "2014-04-17"
title:       "Execute Python Commands Over SSH"
# description: "Execute a remote command or script over shh on a single or group or machines using Python"
tags:        [ "python"]
topics:      [ "Scripts"]
slug:        "ssh-script"
---

This script can be used to execute a single command or a script that is located on a remote machine.  The first argument is a text file containing a list of servers that you wish to execute the command on.  The next arguments are for the ssh user/password.  You can also hard code these into a file to avoid typing them or for allowing you to execute this script automatically. The only dependency is pexpect and this will run in any Python version > 2.4. The latest version can be found on [GitHub][1]

{{< highlight python >}}

import pexpect
import sys

# the file containing the list of servers to log into
input_file = sys.argv[1]

# The login creds
user = sys.argv[2] 
password= sys.argv[3]

def ssh_command (user, host, password, command):

  """This runs a command on the remote host."""
  print " I am logging into", host

  ssh_newkey = 'Are you sure you want to continue connecting'
  child = pexpect.spawn('ssh -l %s %s %s'%(user, host, command))
  i = child.expect([pexpect.TIMEOUT, ssh_newkey, 'password: '])
  if i == 0: # Timeout
    print('ERROR!')
    print('SSH could not login. Here is what SSH said:')
    print(child.before, child.after)
    return None
  if i == 1: # SSH does not have the public key. Just accept it.
    child.sendline ('yes')
    child.expect ('password: ')
    i = child.expect([pexpect.TIMEOUT, 'password: '])
    if i == 0: # Timeout
      print('9ERROR!')
      print('SSH could not login. Here is what SSH said:')
      print(child.before, child.after)
      return None       
  child.sendline(password)
  return child

def main():

  f = open(input_file, "r")
  server_list = f.readlines()

  for server in server_list:
    child = ssh_command (user, server, password, 'script.sh')
    child.expect(pexpect.EOF)
    output = child.before

if __name__ == "__main__":
  main()
  
{{< /highlight >}}

[1]: https://gist.github.com/mattyjones/10666342
