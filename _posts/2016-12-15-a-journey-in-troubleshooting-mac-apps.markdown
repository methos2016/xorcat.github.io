---
author: XORcat
comments: true
date: 2016-12-15 01:30:35+00:00
layout: single
slug: a-journey-in-troubleshooting-mac-apps
title: A Journey in Troubleshooting Mac Apps
wordpress_id: 399
categories:
- Tech
tags:
- '10.12'
- app store
- mac
- macOS
- os x
- Sierra
- tech
- troubleshooting
- tweetbot
---

Hey there!

Long time, no see.

Today I decided to try and tackle an issue I’ve been having for a little while on my Mac.

The issue was with Tweetbot from the Mac App Store, which was previously working, but seemingly out of the blue stopped launching one day a few weeks ago. My Mac is running 10.12.1, and has recently had a re-install done, so is reasonably clean and stable.

When trying to launch it I would see it running for a split second in “Force Quit Applications” and then disappear, only to reappear about a second later for a split second, over and over again. I never saw a window pop up, but if the dock was showing I would see the dock expand a tiny bit as if the Tweetbot icon was being added, and then shrink again as it quit.

With the assistance of Apple’s own support team, I found the issue and got it fixed. I figured this might be a good time to help others if they’re in a similar situation.

I don’t believe this issue was specific to Tweetbot, although I may be wrong, as I don’t know the complete technical details of what was going wrong, just how I managed to fix it.

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


_Note: If running the above nvram command, you will need to run_

    
    sudo nvram boot-args=""


_after you’re finished in safe mode before rebooting again, otherwise you’ll be stuck in safe mode forever!_

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

In order to disable any of these items, you can either _sudo rm file.plist_ them, or (the safer way) you could simply

    
    mkdir ~/Desktop/tempLaunchFiles; sudo mv file.plist ~/Desktop/tempLaunchFiles


to move the suspect file onto your desktop. If you find after doing this and rebooting that your issue isn’t fixed, you can pretty safely restore that .plist to where it came from (or leave it out if you don’t want it!)

My biggest suggestion with troubleshooting is to take small steps. Do not go deleting all of the plist files in one go, otherwise you won’t know which app was causing the issue in the first place! Disable things, and take steps one by one before re-testing.

Let me know if this helped you. Also let me know if you’ve got an issue that this didn’t assist with and you think I might be able to help further troubleshoot it.
I’m always around on Twitter [@XORcat](https://twitter.com/xorcat), or on email at [xorcat@riseup.net](mailto:xorcat@riseup.net)

Thanks!
XORcat
