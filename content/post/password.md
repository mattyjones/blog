---
date:        "2013-05-05"
title:       "The Plaintext Truth About Password Storage Vulnerabilties"
# description: "Your super secure password, is not that super and mostly could be cracked by the time you finish reading this line"
tags:        ["python", "scrypt", "bcrypt"]
topics:      ["Security"]
slug:        "password"
---

It seems every few months a major site has their user database compromised one way or another. Gwaker, LinkedIn, and last.fm among others have been attacked, having their user's login credentials and other data posted online for the world to see. How this happened is an after thought, with near certainty I can say that any information stored on a computer connected to a network will at some point be compromised. This is just the way of life for security professionals, we need to defend against that using proper security protocols, secure coding practices, and constant vigilance, but what happens in the aftermath of a breach? An attacker has managed to download the entire database from your website, all those annoying captcha's, time-delayed logins, and the fancy two-factor authentication are no good now, it is just your data and an attacker that has time and resources on their side, or do they.

With any luck you have sufficient protections in place that will buy you at least some time with which to react to the break-in. The obvious goal would be to prevent the attacker from decrypting any of the credentials in the database, baring that your security measures will hopefully buy you enough time to notify customers of the breach, take precautions to contain the damage, and implement new measures to prevent this in the future. So what security methods should you be using to keep those login credentials safe and why is it such a big deal) so you should be fine, right? Yes...no...depends, there is no easy answer to this so lets start with what a hash actually is.

The Hash
--------

In its most basic form a hash, also called a digest, is a long string of hex characters output from a cryptographic hashing algorithm such as MD5 or SHA-1.

{{< highlight python >}}

import hashlib

password = "mypassword"

def MakeHash(x):
  h = hashlib.sha1(x)
  return h.hexdigest()

d = MakeHash(password)

print d

~$ 91dfd9ddb4198affc5c194cd8ce6d338fde470e2

{{< /highlight >}}

This is not meant to be implemented but merely for an example.  You could run this yourself by copying it into an editor, saving, and launching it with a python version greater than 2.7.  You can see how quickly it executes.  This is the point with SHA-1 and MD5, they are designed, not to store passwords, but to verify the integrity of a file.  Which brings us to an important lesson, the algorithm should work at its best when used on the hardware it was designed for but receive no gain or work poorly in any other environment.  Both SHA-1 and MD5 work great on commodity hardware and if an attacker were to build a machine for the sole purpose of cracking passwords they would be able to wreak havoc on keyspaces up to 6 characters without even breaking a sweat.

Time for a little math lesson.  A typical password may contain 26 letters and 10 numbers which when we account for lowercase and uppercase gives use a total character set of 62.  We use an exponent to represent the size of the key space, in this case 6 which gives us a total of 5 billion possible hashes.  On a typical system it would take about an hour and a half to brute-force the entire keyspace, on a purpose-built machine it will take only a few minutes, if that.

[Project Erebus][1] v2.5 by d3ad0ne, is a $12000 beast, the heart of which is eight AMD Radeon HD7970 GPU cards. Running version 0.10 of oclHashcat-lite, it needed only 12 hours to brute force the entire keyspace for any eight-character password containing upper-case and lower-case letters, digits or symbols. Doing the math that is 96 characters with an exponent of 8, which figures out to 7.2 quadrillion hashes. Keep in mind that this was for the entire keyspace, the vast majority of passwords likely fell within the first 2 hours. This is what your database could be facing and with systems like this being created it won't take long for it to crack under pressure.

Going to larger more complex passwords is not the answer, this will only work until the next advancement in hardware comes along at best, at worst your users will simply type the same word over and over or fill the keyspace up with 0-N numbers. This is just as bad and can actually be broken eaiser using mask attacks or my favorite, hybrid attacks.  I will go into detail in another post, for now just know that systems like this are out there and more common than you think.  The solution, if you can call it that is to start taking password security seriously, starting with throwing a little salt at the attacker, and maybe some pepper as well.

The Pot Of Gold At The End Of The Rainbow
-----------------------------------------

When people talk about pre-compiled tables used to crack passwords they are generally referring to two types, the simple two column lookup tables and the much more common rainbow table.  The simple tables are just what they sound like, a hash calculated using a specific algorithm on one side and the cleartext on the other side.  When using these tables the attacker is freed from having to do any computation at all.  They merely search and compare the hashes, thus common passwords such as password1 fall instantly.  When combined with other attack methods such as a hybrid attack, which involves rules that alter the tables in common ways, it becomes very simple and fast to pull out many of the most common passwords as multiple people may have used the same password but the attacker only has to search once.  This fact alone makes tables very cost-effective from a time-cost standpoint.

While these tables work well for shorter passwords, when the keyspace grows they become far to large.  Rainbow tables were created to help alleviate some of the unwieldiness.  They use a type of hash chain to cut down on what the table has to store.  These only work up to a given keyspace as well and really came into widespread use as RAM prices dropped, allowing attackers to store much of the table in active memory.  Once the key space gets to about nine these become too large as well.  This is vastly simplifying the ideas and math behind tables but it gives you an idea about what they are and how they function in a broad sense.  I will go into much greater detail in a later article but for most purposes due to the advances in GPU and FPGA hardware based attacks tables are not time-cost effective anymore except under very specific conditions involving the keyspace and algorithm the attacker is working against.

Season It With Some Salt And Pepper or The End Of The Rainbow
-------------------------------------------------------------

People love to talk about salting passwords and making sure to use a good salt but what do they really mean? A salt is nothing more than a string that is added to the password before it is hashed, generally at the beginning but you could put it at the end it makes no difference as long as you are consistent. There is really nothing complex about it, the main reason to salt the password is to prevent attacks with pre-compiled tables, including rainbow tables, which has the added result of preventing attack cost sharing.  As tables became more common in the early and mid 2000's salting the password became a hard requirement.

The premise behind a salt is that even a slight change in the password will alter the hash in a major way. This is what is known as the avalanche effect, producing digests that are very different even when the input is only slightly altered, no longer will password1 be hashed to the same value, thus creating more work for the attacker.

{{< highlight python >}}

import os, hashlib, base64

password = "mypassword"
salt = base64.b64encode(os.urandom(64))

def MakeSaltedHash(p,s):
  h = hashlib.sha1(s + p)
  return h.hexdigest()

d = MakeSaltedHash(password, salt)

print d

~$ b6bc00f55aad0961ed0d7bcc1262dba9ab28f00b

{{< /highlight >}}

This will now salt the password with a large enough string to prevent what is known as salt collision, when two password hashes use the same salt. This is a rather excessively large salt but some password files can have over 100,000 entries so this may be necessary to counter what is known as the [birthday problem][2]. A more reasonable value for the salt would be 32bits which would require nearly 4 billion separate tables and roughly 50,000 outputs before you encountered a 25% collision rate. In the above example with 100,000 entries and a 32bit salt the probability of a random collision would be about 70%. Please do not do silly things like hash the username, UID or some other known value as a salt either.  This is a waste and opens the hash to various attack vectors, to be effective the salt needs to be a cryptographically random string and yes there is a difference between a pseudo-random string and a cryptographically random string. There is no such thing as a truly random number but there are viable ways to come close. According to the official Python [documentation][3] os.urandom is suitable for this type of application but to be certain other references should also be sought out.

The salt is stored with the password in the format of $x$salt$hash where x is the algorithm used, which contrary to initial perception, does not present any more of a risk as it is not meant to be secret, but merely to make the decryption process more time expensive for the attacker.  For those that want more protection of this sort, we have pepper.  Pepper is nothing more than a string stored outside the database, generally in source code, that is also used in the hashing process.  It must be noted though that if the attacker gains local access to the server then this method in null.  This is only a feasible method if the attacker only manages to dump the database and even then the jury is still out whether this is really worth the extra effort.  The other concern is that when applying pepper, by its very nature, you now have applied a "key" and as such that key must be protected and you are also starting to get into HMAC, hash message authentication code.

Stretch Those Legs For The Long Road Ahead
------------------------------------------

Salting is only half the battle, as stated before you have to make the attack cost more when run in non-native environments.  This involves a process known as key streching, where you hash the salt and password, rinse, repeat.  There are several ways to do this, I did it this way for the sake of simplicity.

  {{< highlight python >}}

import os, hashlib, base64

password = "mypassword"
salt = base64.b64encode(os.urandom(64))
iterations = "10000"

def MakeSaltedHash(p, s, i):
  h = haslib.sha1(s + p)
  while iterations > 0:
    h = hashlib.sha1(h.hexdigest() + p + s)
    iterations = iterations - 1
  return h.hexdigest()

d = MakeSaltedHash(password, salt, iterations)

~# 10adf3f3ec724dd4c3acf9aca51c29e4bcb35d04

{{< /highlight >}}

This will now perform the hashing process 10,000 times which makes it much more difficult to implement effectively on GPU's.  This is not a perfect solution and I stress again should not be implemented as written because there is a attack vector known as a  transferable state attack which can reduce the computational time necessary by 80% when using SHA-256.  The correct way to implement this would be to use Adaptive Key Derivation Functions such as  bcrypt, which uses a work factor and a block cipher called blowfish which along with the work factor means it is slow as hell in relative terms.

{{< highlight python >}}

import bcrypt

password = "mypassword"

def MakeBcryptHash(p):
  salt = bcrypt.gensalt()
  h = bcrypt.hashpw(salt, p)
return h

d = MakeBcryptHash(password)

~# 708fdc2140b61fbc2100c2c759f76fbce207b58f9aae80db850fc7ab266e1a7e0dc5361c6a47d95f663efae78e0702b41423f2bd355445181dd6438ae846f7fb

{{< /highlight >}}

So we have successfully beaten tables with salt and countered GPU accelerated systems with key stretching, all that is left are the FPGA's.  To counter them we need to move to memory-bound cryptographic functions.

We Need More Memory
-------------------

The other major hardware based attack comes from FPGAs, Field Programmable Logic Arrays.  These work similar to GPU's in that they can do many simple calculations very quickly but unlike GPU's they have dedicated on-board memory.  The only way to counter them is to use memory-bound functions.  These are math functions designed to use lots of memory in non-contiguous blocks.  While FPGA's have on-board memory they are not capable of handling the requirements of these functions whose computational time is dependent upon access to often large amounts of memory.  One choice that is gaining in popularity is scrypt.  While it is not in widespread use due to is relatively untested nature it is starting to slowly gain traction. It contains a work factor like bcrypt but also a memory factor that allows the administrator to dictate memory usage. This combined with other features do the most complete job of defeating many of the available attack vectors. The old saying still applies though, "Garbage in, garbage out". If the tools are not used properly and the users don't at least attempt to pick a secure password no amount of math mumbo jumbo will protect your data.

[1]: http://ob-security.info/?p=546
[2]: http://en.wikipedia.org/wiki/Birthday_problem
[3]: http://docs.python.org/dev/library/os.html?highlight=urandom#os.urandom