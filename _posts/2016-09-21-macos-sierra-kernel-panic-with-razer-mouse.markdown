---
author: XORcat
comments: true
date: 2016-09-21 07:05:37+00:00
layout: single
slug: macos-sierra-kernel-panic-with-razer-mouse
title: MacOS Sierra Kernel Panic with Razer Mouse
excerpt: Damn 3rd party kexts!
categories:
- Tech
tags:
- '10.12'
- com.razer.common.razerhid
- crash
- kernel panic
- macOS
- orochi
- panic
- razer
- razer kext
- razer synapse
- remove razer kext
- Sierra
- Synapse
header:
  teaser: "/assets/images/posts/0006-macos-sierra-kernel-panic/sad.jpg"
---

![Sad Mac](/assets/images/posts/0006-macos-sierra-kernel-panic/sad.jpg)


Hey there!

I upgraded my Mac to 10.12 (Sierra) today, and immediately ran into a couple of problems. One was that two of my most important apps for my productivity were broken: [Karabiner and Seil.](https://pqrs.org/index.html.en)

I've used Seil and Karabiner for a long time to come up with a [use for my caps lock key](http://brettterpstra.com/2012/12/08/a-useful-caps-lock-key/), and be able to do more from the keyboard, without lifting my fingers from typing position. Unfortunately, they are broken in 10.12, although the developer is working on compatibility. If you've used Karabiner/Seil before, you should definitely [donate to the developer](https://pqrs.org/osx/karabiner/donation.html.en) to help out!

Anyway, the real reason we're here: the second (and arguably, more severe) problem I ran into straight after updating to Sierra was that after 5 minutes or so of booting, the machine would kernel panic (crash, and display the power icon on next boot, indicating there was a problem).

I restarted, and it did it again, and again, and again...

Before kernel panicking, the machine would start "stuttering".

All motion on the screen would stop for almost a second, and any input sent to the machine during that time was lost, all keypresses and mouse movements made during the pause did not happen, even after regular operation resumed.

Looking through the .panic file generated (and stored in /Library/Logs/DiagnosticReports/), I saw that the last loaded kext (kernel extension) was com.razer.common.razerhid.

I use a Razer Orochi bluetooth mouse.

I turned the mouse off, and rebooted the Mac, and didn't experience any of the above symptoms. The Mac also didn't kernel panic.

I then powered the mouse back on, and after around 5 minutes, the symptoms appeared, and the machine panicked again.

Uninstalling Razer Synapse didn't help, the same kext would show up in the panic, even after uninstalling Synapse.

Ultimately, I got the mouse to work, and not crash the Mac by [re]moving the Razer kext that was being loaded.

In order to do this, you should first uninstall Razer Synapse (/Applications/Utilities/Uninstall Synapse.app), reboot, and then move RazerHid.kext to a temporary directory, and reboot the Mac again.

    
    mkdir ~/tmp
    sudo mv -v /Library/Extensions/RazerHid.kext ~/tmp/


[Razer is working on a fix](https://twitter.com/RazerSupport/status/778471041605644292), but in the meantime, the above should at least get your computer running without crashing every 5 minutes!
