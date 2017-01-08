---
author: XORcat
comments: true
date: 2016-12-19 14:25:35+10:00
layout: single
slug: tshooting-waas
title: Troubleshooting Cisco WAAS with Akamai Connect
excerpt: Sharing the tips and tricks I've picked up from dealing with WAAS and Akamai for over two years.
categories:
- Tech
tags:
- cisco
- waas
- akamai
- caching
- troubleshooting
header:
  teaser: "/assets/images/posts/0009-tshooting-waas/web-caching.png"
---

![Web Caching](/assets/images/posts/0009-tshooting-waas/web-caching.png)

Hey there!

I'll be honest. I've been working with Cisco WAAS and Akamai Connect for over two years, my experience with reporting on, and troubleshooting the platform has been much less than pleasant.

Because of this, I've picked up on a few things that make my life a little bit easier while troubleshooting WAAS/Akamai Connect. I hope to make your life a little bit easier by sharing these with you, so that you can do more on your own, without having to reach out to (the sometimes slow and frustrating) Cisco TAC.

I'm not going to go into detail of what Cisco WAAS is, because I'm assuming people reading this already know of it (and those who don't know don't have much reason to be here...)

---

*Before we go on, if you're only after some documentation for Akamai Connect (because we all know how hard it is to find), here is a link to [Configuring Cisco WAAS with Akamai Connect](https://www.cisco.com/c/en/us/td/docs/app_ntwk_services/waas/waas/v623/configuration/guide/cnfg/akamai-connect.pdf).*

*I'm going to try not to repeat anything that is already easily accessible in documentation I've linked, so if you're after something that I haven't covered, try checking out the documentation.*

---

**Akamai Connect**, however, is a product that has been baked into WAAS for the last few major versions. Akamai Connect works with WAAS to act as an in-branch content cache, with the advantage of using Akamai's "Over the Top" (OTT) delivery technology. Akamai Connect takes advantage of Akamai's partnerships with content providers such as Apple (for app/OS update delivery) and Google (for YouTube video delivery) in order to provide a greater user experience by storing content locally, and serving it directly to the client while it is still fresh.

*[OTT]: Over The Top

Just to be clear, so you know what I've experienced, I have:

* deployed > 300 WAAS devices, all with Akamai Connect;
* which are all running on VMware ESXi;
* running WAAS software v5.5.5a;
* and Internet access is only available via an authenticated proxy.

Akamai Connect doesn't really have much in the way of an interface which the administrator can directly manage the product through. All of the configuration of Akamai Connect is done through WAAS.

_Akamai Connect is a product from Akamai and Cisco. When I say Akamai Connect, I am speaking only of this product, not of everything produced by the CDN Akamai_

## Connectivity to Akamai Connect

### Connecting the WAAS to Akamai Connect

In order to learn Akamai's current OTT rules and to verify your licensing, the WAAS must connect to Akamai's management servers periodically. Until this has occurred successfully at least once, Akamai Connect will be non-functional, and the WAAS will only cache basic HTTP objects, like a Squid cache would.

The WAAS requires connectivity to the host `amg.terra.akamai.com` over SSL/TLS on port `443`.

There is also a requirement for connectivity to an address ending in `.omni.akamaiapis.net` over SSL/TLS on port `443`. This hostname is provided in the Akamai Connect license file that you install to the CM when first configuring and enabling Akamai Connect. As far as I can tell, this host does not change, but I cannot confirm that.

This connectivity does not need to be constant, however, as stated above, Akamai Connect will not do much more than basic object caching until this has succeeded at least once.

Unfortunately as of v6.2, there is still no functionality to configure the WAAS with credentials for it to use with a proxy to access the Akamai AMG server.

My current workaround for this situation is to allow the WAAS Central Manager (CM) direct access through the firewall to anything on `443`, and then configure the edge WAAS to use the CM as a proxy. You could also allow only specific destinatio hosts, however I've found DNS based rules to be unreliable in the past when the destination host uses a dynamic IP address.

### Verifying Connectivity to Akamai Connect

Connectivity to Akamai Connect can be verified with the following command 

### Troubleshooting Connectivity to Akamai Connect

## Other

### Full Disk on CM

Let me know if this helped you. Also let me know if you’ve got an issue that this didn’t assist with and you think I might be able to help further troubleshoot it.
I’m always around on Twitter [@XORcat](https://twitter.com/xorcat), or on email at [xorcat@riseup.net](mailto:xorcat@riseup.net)

Thanks!
XORcat

*[AMG]: Akamai Management Gateway

*[CM]: Central Manager