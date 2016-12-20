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
  teaser: "/assets/images/posts/0008-tshooting-waas/web-caching.png"
---

![Web Caching](/assets/images/posts/0008-tshooting-waas/web-caching.png)

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

**Connecting the WAAS to Akamai Connect**

In order to learn Akamai's current OTT rules and to verify your licensing, the WAAS must connect to Akamai's management servers periodically. Until this has occurred successfully at least once, Akamai Connect will be non-functional, and the WAAS will only cache basic HTTP objects, like a Squid cache would.

The WAAS requires connectivity to the host `amg.terra.akamai.com` over SSL/TLS on port `443`.

There is also a requirement for connectivity to an address ending in `.omni.akamaiapis.net` over SSL/TLS on port `443`. This hostname is provided in the Akamai Connect license file that you install to the CM when first configuring and enabling Akamai Connect. As far as I can tell, this host does not change, but I cannot confirm that.

This connectivity does not need to be constant, however, as stated above, Akamai Connect will not do much until this has succeeded at least once.

Unfortunately as of v6.2, there is still no functionality to configure the WAAS with credentials for it to use with a proxy to access the Akamai AMG server.

*[AMG]: Akamai Management Gateway

My current workaround for this situation is to allow the WAAS Central Manager (CM) direct access through the firewall to anything on `443`, and then configure the edge WAAS to use the CM as a proxy. You could also allow only specific hosts, however I've found DNS based rules to be unreliable in the past when the destination host uses a dynamic IP address.

*[CM]: Central Manager

**Symptoms**

As I said above, the app would show up in the dock and force quit apps for a split second before disappearing. No other window would appear.
Occasionally I would get the following window pop up, asking me to enter my Apple ID as the app was purchased on another Mac.
(which it wasn’t… 😟)

[![](https://pbs.twimg.com/media/Cx7Ly3EXcAEagCx.jpg)](https://pbs.twimg.com/media/Cx7Ly3EXcAEagCx.jpg)

[These](http://pastebin.com/s0xa4CPm) were some of the log messages that I found in Console.app, but they didn’t point me in any kind of useful direction.

**Low hanging fruit**

The first thing I tried was to delete the app, restart, and re-install it from the App Store.

That was a great idea, but unfortunately, it didn’t work for me.

**Creating a new MacOS user profile**

Next step was to create a new user account (for this, I just created a dummy “test” account that was not very long lived).

The idea here is to rule out user-specific settings or apps that launch when a specific user logs in.

If there is an application running that is messing with this broken app, testing in a new user account may provide a cleaner environment where the intrusive app doesn’t launch.

Also a great idea, also didn’t work for me.

**Booting into [Safe Mode](https://support.apple.com/en-us/HT201262)**

To be completely honest, I didn’t even realise MacOS even had a safe mode until I spoke to the support engineer at Apple.

As this link suggests, safe mode stops startup items and login items from running, only loads required kernel extensions, disables user-installed fonts, and does a couple of disk checks.

Effectively, this gives you the cleanest possible environment to do your testing in. If you can’t get your app to work here, either your app is corrupted/broken, or you have some more deeply rooted system issues.

Oh, and you can launch safe mode by holding shift while your machine is first booting. Unfortunately, for some reason, that didn’t work for me, and I had to run the following from Terminal before rebooting (as stated on [Apple's Safe Mode page](https://support.apple.com/en-us/HT201262)):
    
    sudo nvram boot-args="-x”

*Note: If running the above nvram command, you will need to run_ `sudo nvram boot-args=""` _after you’re finished in safe mode before rebooting again, otherwise you’ll be stuck in safe mode forever!*

So, I tried launching Tweetbot from safe mode, and it worked straight away!

Nice! Then I tried rebooting into regular mode again, but unfortunately the issue still remained once back in regular mode. Safe mode didn’t fix my issue, but the app works when in safe mode.

Maybe one of my startup/login items/daemons is screwing with me?

**Disabling Login/Startup Items/Daemons**

This is where I found my solution. It turns out that I had (purposefully) instructed a script to run each time the OS booted, which would randomise the MAC address of my wireless adapter. I had not had any issues with that, and it had been running successfully, until I realised it was the culprit behind Tweetbot not launching.

There are a few places that programs that want to launch automatically at different times of the boot/login process can live.

User-specific “Login Items” that launch upon a specific user logging in can be found/disabled from System Preferences —> Users & Groups —> Your Username —> Login Items.

Clicking on the minus icon will remove the selected app from Login Items, and next time you log in it should not launch itself.

The other places these things can hide are in the following directories:

    
    /Library/LaunchDaemons
    /Library/LaunchAgents
    /Users/yourusername/Library/LaunchAgents


In these directories are preference (.plist) files which describe a program that should be run at either boot (LaunchDaemons) or login (LaunchAgents). The user-specific LaunchAgents directory will only affect that user.

These plist files generally have quite user-friendly names so that you can find what you’re looking for fairly easily.

For example:
    
    net.tunnelblick.tunnelblick.LaunchAtLogin.plist

looks like this:
    
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
         <key>Label</key>
         <string>net.tunnelblick.tunnelblick.LaunchAtLogin</string>
         <key>ProgramArguments</key>
         <array>
              <string>/Applications/Tunnelblick.app/Contents/Resources/launchAtLogin.sh</string>
         </array>
         <key>LimitLoadToSessionType</key>
         <string>Aqua</string>
         <key>RunAtLoad</key>
         <true/>
         <key>ExitTimeOut</key>
         <integer>0</integer>
         <key>ProcessType</key>
         <string>Interactive</string>
    </dict>
    </plist>

This is the user-specific LaunchAgent plist file for Tunnelblick, the VPN software that I use.
    
    /Applications/Tunnelblick.app/Contents/Resources/launchAtLogin.sh

is the command that will be run when I log in.

In order to disable any of these items, you can either `sudo rm file.plist` them, or (the safer way) you could simply
    
    mkdir ~/Desktop/tempLaunchFiles; sudo mv file.plist ~/Desktop/tempLaunchFiles

to move the suspect file onto your desktop. If you find after doing this and rebooting that your issue isn’t fixed, you can pretty safely restore that .plist to where it came from (or leave it out if you don’t want it!)

My biggest suggestion with troubleshooting is to take small steps. Do not go deleting all of the plist files in one go, otherwise you won’t know which app was causing the issue in the first place! Disable things, and take steps one by one before re-testing.

Let me know if this helped you. Also let me know if you’ve got an issue that this didn’t assist with and you think I might be able to help further troubleshoot it.
I’m always around on Twitter [@XORcat](https://twitter.com/xorcat), or on email at [xorcat@riseup.net](mailto:xorcat@riseup.net)

Thanks!
XORcat

