---
author: XORcat
comments: true
date: 2016-12-20 22:55:04+10:00
layout: single
slug: telstra-tpb-block
title: Telstra Has "Blocked" ThePirateBay.org
excerpt: How this looks, and what you can do about it.
categories:
- Tech
tags:
- telstra
- blocked
- piracy
- thepiratebay
- torrentz
- torrents
- torrenthound
- isohunt
- solarmovie
- bypass
- dns poison
- bypass block
header:
  teaser: "/assets/images/posts/0008-telstra-tpb-block/message.png"
---

Hey there!

I've just noticed that today, Telstra (the biggest ISP in Australia) has started to block ThePirateBay, and a number of other torrent file sharing sites, due to a ruling made in the Federal Court in Sydney last week.

The message Telstra serves you when visiting these sites is terribly written, but gets the point across.

![Block Message](/assets/images/posts/0008-telstra-tpb-block/message.png)

The full list of blocked domains can be found [here](https://s115a.com/sites/blocked), and more information about the case [here](https://s115a.com/news/update/28/the-blocks-begin-the-pirate-bay-torrentz-torrenthound-isohunt-and-solarmovie-gone).

This has been expected for a little while now, as [a modification to the copyright act](http://parlinfo.aph.gov.au/parlInfo/download/legislation/billsdgs/3830145/upload_binary/3830145.pdf;fileType=application/pdf) was passed in 2015 to:

> ... enable copyright owners to apply to the Federal Court of Australia for an order requiring a carriage service provider to block access to an online location operated outside Australia that has the primary purpose of infringing copyright or facilitating the infringement of copyright.

[Will Ockenden](https://willockenden.com/) has [a great summary](https://s115a.com/about) of the history of this legislation on his website dedicated to the issue, [s.115a](https://s115a.com/).

---

Let's be real, you're most likely here to try and find a way around this block. Luckily for you, it's only a minor inconvenience to you, and it also [not illegal to bypass the block](http://www.lifehacker.com.au/2016/12/is-it-legal-to-access-isp-blocked-websites/), although that doesn't mean the torrenting of copyrighted material you participate in is not illegal.

Regardless, if you need, or want to access ThePirateBay, Torrentz.eu, or other sites blocked by Telstra, here is the deal.

Telstra has implemented these blocks using a method known as ["DNS Spoofing/Poisoning"](https://en.wikipedia.org/wiki/DNS_spoofing). This means that when your computer asks Telstra's DNS server for the address of thepiratebay.org, it responds with the address of one of Telstra's servers, which serves up a message telling you that the page is blocked.

A really easy way to get around this, is to tell your computer not to use Telstra's DNS servers (there's nothing forcing you to!), and instead to use another server, such as Google's 8.8.8.8/8.8.4.4, or OpenDNS' servers.

Google has a [comprehensive guide](https://developers.google.com/speed/public-dns/docs/using?csw=1) on how to set your computer to use their DNS servers instead of Telstra's.

Essentially, Telstra hasn't blocked access to these sites at all, they have simply stopped telling you how to reach them. If you ask around, plenty of other DNS servers will tell you how to get there, and Telstra will happily route your packets to their destination.

---

On a side note, it looks like Telstra had prepared for this long ago. Their servers that host the "Content Denied" message first appeared back in November, as shown here by Shodan:

![Shodan First Seen](/assets/images/posts/0008-telstra-tpb-block/shodan-first-seen.png)

---

I hope this has helped you get access to whatever perfectly legal torrents you need to get access to.

If you'd like to get in contact, I’m always around on Twitter [@XORcat](https://twitter.com/xorcat), or on email at [xorcat@riseup.net](mailto:xorcat@riseup.net).

Thanks!

XORcat

