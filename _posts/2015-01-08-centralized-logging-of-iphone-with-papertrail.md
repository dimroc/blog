---
layout: post
title: "Centralized Logging of iPhone apps with Papertrail"
date: "Thu Jan 08 15:13:38 -0500 2015"
tags: ios swift
---

I use [Papertrail](https://papertrailapp.com/) for all my server logging needs, and I love it. I'm almost always bullish on the use of third party
services to offload work from a product dev team. Not too many people want to spend time working on log drains, me included.

When doing iPhone development, working with logs outside of XCode is clumsy at best and usually nonexistent. And when your beta is in the wild, that's not even an option
because you're unable to round up all your testers' devices.

Don't worry, I'm here to tell you your iOS app can be configured to automatically upload logs to [Papertrail](http://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-ios-or-os-x-apps/)
as described [here](http://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-ios-or-os-x-apps/) within 15 minutes.

![Papertrail for iPhone](/public/images/PapertrailForIPhone.png)

<!--more-->

###Status Quo
The standard solution for all this are Crash Reports via Apple Connect, Crashlytics, Parse, etc. But these only take you so far because sometimes you need **more
than a call stack.** These logs will complement the crash report.

###Reference Material
- [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack) is a pod that enables one to easily and asynchronously carry logs from the device to a destination, whether it be a terminal, a file, or a service. It's the keystone to this arch, if you get what I'm saying.
- PapertrailLumberjack allows one to transfer the logs from your iPhone to the Papertrail service. So, as you can see, a lot of the work is done for us.
- Unfortunately, [the instructions here](http://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-ios-or-os-x-apps/) are only talking about Objective C,
and they refer to an outdated CocoaLumberjack.

###Explanation of Solution
1. The PapertrailLumberjack pod doesn't support the current CocoaLumberjack.<br>

    No worries. [There is a patched PaperTrailLumberjack that supports CocoaLumberjack 2.x](https://bitbucket.org/luisrecuenco/papertraillumberjack).

2. PapertrailLumberjack incompatible with Swift CocoaLumberjack.<br>

    I never did get the Swift version of CocoaLumberjack working alongside the Obj C version of PapertrailLumberjack (there's a nightmare waiting to happen when converting ObjC Macros to Swift)
    so I'm using the ObjC version with [my own Swift wrapper](https://gist.github.com/dimroc/2aef1b6b1e391f0085d2) as you can see below.

As Swift and CocoaPods mature, I'm sure PaperTrailLumberjack will just work out of the box. Until then, I've got you.

###Steps:

1. Add CocoaLumberjack 2.x and the patched PaperTrailLumberjack to your Podfile, then `pod install`.

    ```ruby
    pod 'CocoaLumberjack', git: 'git@github.com:dimroc/CocoaLumberjack.git'
    pod 'PaperTrailLumberjack', git: 'https://bitbucket.org/luisrecuenco/papertraillumberjack.git'
    ```

2. Add the #imports to your bridging header file.

    ```C++
    #import "CocoaLumberjack.h"
    #import "RMPaperTrailLogger.h"
    ```

3. For Swift, add my DDLogHelper to your project (copy pasta).

    {% gist dimroc/2aef1b6b1e391f0085d2 %}

4. Configure DDLog and PapertrailLumberjack with your settings.

    ```swift
    class func launch() { // Or any initializer method
      let paperTrailLogger = RMPaperTrailLogger.sharedInstance()
      paperTrailLogger.host = "MyHost.papertrailapp.com"
      paperTrailLogger.port = myportnumber

      DDLog.addLogger(paperTrailLogger)
      DDLog.addLogger(DDASLLogger.sharedInstance())
      DDLog.addLogger(DDTTYLogger.sharedInstance())
    }
    ```

5. Log away.

    ```swift
    DDLogHelper.debug("Launching Papertrail logging for My App")
    ```

###Conclusion
Compared to what I was doing before, I'm as happy as can be. And this was all free. I'm still waiting to see what the impact on battery life and performance is. If you have any experience with this, drop me a line below.
