---
author: XORcat
comments: true
date: 2016-08-19 02:56:37+00:00
layout: single
slug: equation-group-crashing-asas-follow-up
title: Equation Group - Crashing ASAs + Follow Up
wordpress_id: 377
categories:
- InfoSec
tags:
- asa
- ASA Exploit
- cisco
- crash asa
- EquationGroup
- ExtraBacon
- InfoSec
- security
- ShadowBrokers
- tech
---

Hi there!


It's been a crazy couple of days, after I wrote my post about breaking into the 8.4(2) ASA with ExtraBacon, others have tested EpicBanana, and various other tools from the dump.

[Cisco has also written their own post](https://blogs.cisco.com/security/shadow-brokers) about EpicBanana and ExtraBacon, confirming the vulnerabilities exist, creating CVEs for them (CVE-2016-6367 and CVE-2016-6366 respectively).

Unfortunately, there is not a software fix for CVE-2016-6366 (ExtraBacon) yet, but Cisco said it is being currently worked on.<!-- more -->

[There has also been mention](https://musalbas.com/2016/08/18/equation-group-benigncertain.html) (by Mustafa Al-Bassam) of a tool in the dump called "BenignCertain", which appears to send a crafted IKE packet to a Cisco PIX v5.2 through to 6.4(4), parse the response, and find the pre-shared key configured for the VPN tunnel.... I've not been able to get it to work on ASA 8.4(2) (ASA is the successor to PIX), and unfortunately I haven't been able to get my hands on a working PIX to test it out properly.

Another interesting thing that Cisco was able to do with ExtraBacon was crash an ASA that isn't running the versions of software that ExtraBacon supports.

The extremely easy way to do this is to modify the part of ExtraBacon that checks the ASA software version, and tell it to go ahead and send a payload, even if the version isn't supported by the tool.

In my lab, I told ExtraBacon to send the ASA 8.4(4) payload if the ASA it was talking to was not one of the supported versions.

I built an ASA running 9.1(5) (software released in 2014) in my GNS3 lab, aimed my modified ExtraBacon at it, and....

![exba-crash](https://xorcat.net/wp-content/uploads/2016/08/exba-crash.png)

The firewall was dead instantly, no traffic being passed. That's scary, right? Right.

Even worse though, is that the firewall didn't even reboot properly! It just sat there like this (I waited over 10 minutes, but it did nothing):

![exba-crash-noreload](https://xorcat.net/wp-content/uploads/2016/08/exba-crash-noreload.png)

It's possible that my GNS3 ASA is behaving differently to how a real physical ASA would, but I can't tell for sure whether that is the case.

Note: This does not work unless the SNMP community string is known. If you try and use an incorrect string, the ASA will just ignore you.

What I do know is that you do not want to be allowing this to happen to your firewall.

In the post Cisco wrote about EpicBanana and ExtraBacon, they have provided Snort rules (3:39885) and Cisco IPS signatures (7655-0) that detect ExtraBacon activity, and are able to stop it.

If you do not have an IPS that can stop this attack, you should definitely look at tightening up the security around your SNMP community string, and review which addresses and interfaces are allowed to talk SNMP to your firewall. Or, if you have to, disable SNMP altogether on your firewall (`no snmp-server enable`) until a fix is released.

[@XORcat](https://twitter.com/xorcat)
