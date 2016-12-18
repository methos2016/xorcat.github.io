---
author: XORcat
comments: true
date: 2016-07-31 01:55:59+00:00
layout: post
link: https://xorcat.net/2016/07/31/what-i-learned-from-the-wall-part-1/
slug: what-i-learned-from-the-wall-part-1
title: What I Learned From "The Wall Part 1"
wordpress_id: 35
categories:
- InfoSec
tags:
- InfoSec
- tech
- vulnhub
---

The Wall, as well as being a classic two part album by Pink Floyd, is a vulnerable VM series created by [@Xerubus](https://twitter.com/xerubus) and hosted on [VulnHub](https://www.vulnhub.com/).

I was not able to complete "[The Wall: 1](https://www.vulnhub.com/entry/the-wall-1,130/)" completely on my own. I made it up to gaining SSH access to the box on my own, but got stuck there and essentially gave up, leaning on [Xerubus' own walkthrough of the VM](http://www.mogozobo.com/?p=2848) for the remainder of the challenge.

I am still only getting started with attacking VulnHub VMs, but I'm not too fresh to know that looking back I should have persisted more before resorting to a walkthrough. If you're here looking for answers, I would highly recommend sleeping on it, and coming back to try a different avenue. If you've done that repeatedly, and you're still stuck, maybe my post can help without completely spoiling the solution for you.

I figured that because I gave up before I should have, the least I could do was to write a post about what I learned from the VM; both to help it sink in, and to give others some tips if they are stuck.

Apart from including methods you can use to solve the challenge, I'm going to keep this post rather vague, and not directly tell you what steps to take at what point in the VM.

**This is NOT a complete walkthrough!**


# 1 - Listen


Port scans are not everything, sometimes you need to listen for traffic to or from your target to find something interesting.

Obviously, the easy tool for this is [tcpdump](http://packetlife.net/media/library/12/tcpdump.pdf).


# **2 - Keep an eye out for hashes**


I ended up with what appeared to me as just 32 hex characters, that were hinted to be some kind of key. I first tried converting them to ASCII using `echo "hash" | xxd -r -p`, only to get a bunch of garbage characters.

I then tried inputting the hex characters as the key (using many different methods), still to no avail.

After getting frustrated, I inputted the characters into Google, and the first link was the [ISC's reverse hash calculator](https://isc.sans.edu/tools/reversehash.html). Inputting the characters there gave me the real key to use.

After reading Xerubus' walkthrough once I got further into the challenge showed me a useful tool to use if you think something might be a hash (or just can't figure out what it is).

    
    root@zazen:~# hash-identifier
       #########################################################################
       #     __  __                     __           ______    _____           #
       #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
       #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
       #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
       #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
       #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
       #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.1 #
       #                                                             By Zion3R #
       #                                                    www.Blackploit.com #
       #                                                   Root@Blackploit.com #
       #########################################################################

    
       -------------------------------------------------------------------------
     HASH: 7ad6c606feae67a5bdac5d1790d994f0
    
    Possible Hashs:
    [+]  MD5
    [+]  Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
    
    Least Possible Hashs:
    [+]  RAdmin v2.x
    [+]  NTLM
    [+]  MD4
    [+]  MD2
    [+]  MD5(HMAC)
    [+]  MD4(HMAC)
    [+]  MD2(HMAC)
    [+]  MD5(HMAC(Wordpress))
    [+]  Haval-128
    [+]  Haval-128(HMAC)
    [+]  RipeMD-128
    [+]  RipeMD-128(HMAC)
    [+]  SNEFRU-128
    [+]  SNEFRU-128(HMAC)
    [+]  Tiger-128
    [+]  Tiger-128(HMAC)
    [+]  md5($pass.$salt)
    [+]  md5($salt.$pass)
    [+]  md5($salt.$pass.$salt)
    [+]  md5($salt.$pass.$username)
    [+]  md5($salt.md5($pass))
    [+]  md5($salt.md5($pass))
    [+]  md5($salt.md5($pass.$salt))
    [+]  md5($salt.md5($pass.$salt))
    [+]  md5($salt.md5($salt.$pass))
    [+]  md5($salt.md5(md5($pass).$salt))
    [+]  md5($username.0.$pass)
    [+]  md5($username.LF.$pass)
    [+]  md5($username.md5($pass).$salt)
    [+]  md5(md5($pass))
    [+]  md5(md5($pass).$salt)
    [+]  md5(md5($pass).md5($salt))
    [+]  md5(md5($salt).$pass)
    [+]  md5(md5($salt).md5($pass))
    [+]  md5(md5($username.$pass).$salt)
    [+]  md5(md5(md5($pass)))
    [+]  md5(md5(md5(md5($pass))))
    [+]  md5(md5(md5(md5(md5($pass)))))
    [+]  md5(sha1($pass))
    [+]  md5(sha1(md5($pass)))
    [+]  md5(sha1(md5(sha1($pass))))
    [+]  md5(strtoupper(md5($pass)))


`hash-identifier` identifies what kind of hash that a given string might be.

I've learned that if given a string I can't quite figure out, I should throw it in to hash-identifier to see whether it is some kind of hash that I can attempt to reverse.


# 3 - stegdetect isn't perfect


I had something that I was sure was supposed to contain steganography, but `stegdetect` told me that there was nothing there.

I took off in another direction, ignoring the evidence right in front of me that pointed to there being a hidden message in a file, because I trusted that `stegdetect` knew what it was talking about.

Later, when I found and reversed a hash that I didn't recognise at first, `steghide` showed me that I was right about there being a hidden message, and that `stegdetect` was lying to me.


# 4 - binwalk and scalpel


I had never done any data forensics before, but The Wall introduced me to `binwalk` and `scalpel`, two really easy to use tools for extracting files out of disk image files. I now feel a bit more confident with where to start performing basic data forensics.


# 5 - Enumerate, enumerate, enumerate


Just because a port was closed at some point, doesn't necessarily mean that it's still closed! If you've made progress, or if time has passed, you should definitely run your scans again if possible.


# 6 - Search for setuid (and setgid) files


One of the first things you should do if you're trying to escalate your privileges is to search far and wide for files with the setuid bit set.

Files that are executable and have the setuid bit are run with the uid of the owner of the file, meaning that if there is a way to break out into a shell from that file, you will get the privileges of the owner (or group) of the file.

`find` can help you find setuid (and setgid) binaries with the following command.

    
    find / -perm -4000 -o -perm -2000 -exec ls -ld {} ; 2>/dev/null


The above will find any files with the setuid or setgid bits set, and the below will find any files with BOTH the setuid and setgid bits set.

    
    find / -perm -6000 -exec ls -ld {} ; 2>/dev/null




# 7 - Read background info


Sometimes you'll find information that seems unimportant. You might just find a gem that could help you in the future, or a direct hint to your next steps.


# 8 - Try using login


Not every user is necessarily allowed to log in using a given service (SSH, FTP, telnet). If the users account is enabled, and you have the right credentials, you have the best chance of getting access to the user's account by using the `login` command.


# 9 - Search for (and exploit) insecure binary calls


`strings` can be used to search for binary calls in an executable file (if it is readable). If a binary is called without specifying it's full path, it is possible to trick the program into running something other than what it intended to.

For example, if a setuid program calls `cal` without specifying it's full path, it's possible to manipulate your PATH variable to include a directory (before the directory containing the real `cal`) containing an executable with different behaviour, such as launching a shell.

    
    ln -s /bin/sh /home/user/bin/cal
    export PATH=/home/user/bin:$PATH


Then, simply get the program to run `cal`, and you should end up with a shell!


# 10 - Find recently modified files


If you're stuck, and unsure of the side effects of running a program, `find` can help you find out if something has made changes to files on the system.

    
    find / -mmin -10 -type f 2>/dev/null


The above will show you which files have been modified within the last 10 minutes, which could help you find out whether an important config file, or log file has been modified.


# Summary


Overall, I think the most important thing that I was taught by this challenge was to be persistent, and not give up and use a walkthrough at the first sign of difficulty. As the folks at Offensive Security say: "Try Harder!".

Regardless of giving up, I still learned a bunch from this challenge, and from Xerubus' walkthrough.

Thank you to Xerubus for creating this challenge, and for giving me a great learning experience.

I'd love to hear what you learned from the challenge, and to read your walkthrough if you've created one. The easiest way to contact me is on Twitter, [@XORcat](https://twitter.com/XORcat)
