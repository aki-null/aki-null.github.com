---
layout: post
title: RegexKit on Snow Leopard
tags:
- Objective-C
---


I have been using RegexKit for the Twitter client called [YoruFukurou](http://sites.google.com/site/yorufukurou/) for quite some time now. The framework was essential for the application, because the primary functionality of the app heavily involved the use of regular expressions. I tried other libraries, but only RegexKit satisfied my requirements.

RegexKit seemed to perform just fine on Snow Leopard, but I found a significant bug. The RegexKit was advertised to support Garbage Collection that was introduced in Leopard, but it seemed to crash as soon as RegexKit API was called on Snow Leopard. After some research, I found out that only one line of code was required to be changed to fix the issue. The bug was being tracked on their [official bug tracker](http://sourceforge.net/tracker/?func=detail&aid=2876858&group_id=204582&atid=990188), but the code maintainer seemed to be inactive. I tried to build the framework after applying the fix, but the build failed miserably on Snow Leopard.

Unfortunately, I had to install Leopard on my external HDD to build the framework, because I had no computers running Leopard. You can download the fixed RegexKit framework from [here](http://aki-null.net/misc/RegexKit.zip), so nobody needs to go through the pain of setting up Leopard system to fix the bug.
