---
author: XORcat
comments: true
date: 2016-08-07 02:28:02+00:00
layout: single
slug: trying-to-fool-asa-stateful-ftp-inspection
title: Trying to fool ASA stateful FTP inspection
excerpt: Fool me once...
categories:
- InfoSec
tags:
- asa
- ccna security
- cisco
- InfoSec
- security
- tech
---

Hi there!

I was doing some CCNA Security study, playing with ASAs, and the ability of the firewall to inspect FTP traffic in order to open ports for [FTP passive mode](http://slacksite.com/other/ftp.html) connections.

For example, when I tell my FTP server that I want to connect in passive mode:

    
    PASV
    227 Entering passive mode (10,1,2,3,253,232).


If this FTP session goes through the ASA, and FTP traffic is inspected by the ASA (as it is by default in the `global_policy` policy-map, at least in ASA 9.1(5)21, see below), the ASA will open up the access to the passive port on the FTP server (65000 (`253 * 256 + 232`) in the above):

    
    policy-map global_policy
     class inspection_default
     inspect dns migrated_dns_map_1
     inspect ftp
     inspect h323 h225
     inspect h323 ras
     inspect ip-options
     inspect netbios
     inspect rsh
     inspect rtsp
     inspect skinny
     inspect esmtp
     inspect sqlnet
     inspect sunrpc
     inspect tftp
     inspect sip
     inspect xdmcp


So, what if I, a malicious user on the inside network of a company with a restrictive outbound firewall policy, wants to SSH to a server that is blocked in the firewall policy, but the firewall allows FTP out. Could I craft an FTP server to tell the firewall that the FTP client needs to connect to port 22 for passive mode?

I thought I'd try!

Initially I thought I'd do this using scapy, which I hadn't played with before, so I found [this great, easy to follow basics guide](https://thepacketgeek.com/series/building-network-tools-with-scapy/).

Unfortunately, I got sidetracked while learning that, and thought I might just be able to find an open source FTP server that I could fiddle with instead, which I did: [pyftpdlib](https://github.com/giampaolo/pyftpdlib) by [giampaolo](https://github.com/giampaolo).

I modified the section of pyftpdlib where it sends back the `227 Entering Passive Mode` message, so that it sent back a port number of my choosing.

I then spun up my GNS3 ASA lab. I used a router on the inside of the ASA as the client, my Mac as the FTP server running my modified pyftpdlib server, and the access-list on the inside of the ASA was this:

    
    access-list inside_in extended permit tcp any any eq 21


Everything else will get dropped, which is what I want to test.

Now, my server has been told to say `227 Entering Passive Mode (10,1,2,3,0,22)`, 10.1.2.3 being my Mac's IP address, and `0,22` meaning `0*256+22`, or just port 22.

Alright, let's try it then! From R1:

    
    R1#telnet 10.1.2.3 21
    Trying 10.1.2.3, 21 ... Open
    220 pyftpdlib 1.6.0 ready.
    user user
    331 Username ok, send password.
    pass 12345
    230 Login successful.
    pasv
    
    [Connection to 10.1.2.3 closed by foreign host]


Hmmm, I got disconnected when running PASV? Let's see what the ASA had to say:

    
    %ASA-4-406001: FTP port command low port: 10.1.2.3/22 to 10.1.0.1 on interface outside
    %ASA-4-507003: tcp flow from inside:10.1.0.1/58593 to outside:10.1.2.3/21 terminated by inspection engine, reason - inspector drop reset.
    %ASA-6-302014: Teardown TCP connection 11 for outside:10.1.2.3/21 to inside:10.1.0.1/58593 duration 0:00:18 bytes 141 Flow closed by inspection


Damn! It's on to me!

So I tried a couple of other ports in the passive mode response, it turns out that the ASA will terminate the connection if it sees a passive mode response with a port of 1023 or below.

Okay, what about tricking the ASA into letting me connect to another host, by getting the server to respond with `227 Entering Passive Mode (1,2,3,4,4,0)` (1.2.3.4 on port 1024).

    
    R1#telnet 10.1.2.3 21
    Trying 10.1.2.3, 21 ... Open
    220 pyftpdlib 1.6.0 ready.
    user user
    331 Username ok, send password.
    pass 12345
    230 Login successful.
    pasv
    
    [Connection to 10.1.2.3 closed by foreign host]


Bugger! Foiled again.

Alright ASA, what now?

    
    %ASA-4-406002: FTP port command different address: 10.1.2.3(1.2.3.4) to 10.1.0.1 on interface outside
    %ASA-4-507003: tcp flow from inside:10.1.0.1/28040 to outside:10.1.2.3/21 terminated by inspection engine, reason - inspector drop reset.
    %ASA-6-302014: Teardown TCP connection 13 for outside:10.1.2.3/21 to inside:10.1.0.1/28040 duration 0:00:12 bytes 196 Flow closed by inspection


So, it turns out that if the FTP server responds with a different IP address, or a port less than 1024 for the passive mode connection, the firewall will destroy the FTP connection altogether.

I did, however notice that if the passive connection was opened (the port was above 1023, and the host was the same), I could do whatever I want with that port, and the ASA would just let it happen.

I'm glad to know the ASA is doing it's job (mostly) properly!

If you have any other ideas on things to try, or any questions about what I've done, I can be found on twitter [@XORcat](https://twitter.com/XORcat).

Thanks for reading!
