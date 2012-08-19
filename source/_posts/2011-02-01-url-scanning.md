---
layout: post
title: URL Scanning
tags:
- Objective-C
---


I have spent significant time trying to scan URLs in a string. The most popular way to do this is to use regular expression.

I have been using regex described [here](http://www.din.or.jp/~ohzaki/perl.htm#httpURL) (Japanese) for my Twitter client [YoruFukurou](http://sites.google.com/site/yorufukurou/home-en). This regular expression was very strict, and followed RFC very well.

However, things are changing on the web. IDN (Internationalized Domain Name) is becoming somewhat popular in Japan, and there are web browsers that do not percent encode URLs copied from its address bar (i.e. Safari). The URL would not be matched by the regular expression above, because it doesn’t expect IDN, or Unicode characters in URL.

Since I was building a Twitter client, it needed to be lenient in the way it parsed URLs. After trying [several different regular expressions](http://daringfireball.net/2010/07/improved_regex_for_matching_urls), I have realised that there might be simpler and more effective way of doing this. I have spent 10 seconds to come up with the following regular expression.

`https?://[^\s]*`

It actually worked very well, because most of people would insert an empty character after an URL. This regular expression matched URLs containing Unicode characters properly.

After using this regular expression to scan for URLs in my Twitter client, I have found one case where it would not work properly. In Twitter, people tend to use brackets around URLs instead of empty characters.

e.g.`[http://ja.wikipedia.org/wiki/正規表現]`

The regular expression I have proposed previously would match the URL with the trailing square bracket character. This was not a desirable behaviour.

The following things must be done to parse this properly.
- Detect the starting bracket character
- Detect the end bracket character
- Check whether the end bracket character matches the starting bracket character



After some experiments, I have realised that regular expression is not powerful enough to handle this case properly. (It may be possible, but it is likely to end up with extremely complex regular expression, which can cause performance issues)

My solution was to write my own URL scanner. I have spent about two days writing the initial version of the scanner. The new URL scanner has successfully solved the issue with brackets. The scanner is capable of finding URLs in the following scenarios.

    String: [https://テスト).com]
    URL: https://テスト).com

    String: (http://en.wikipedia.org/wiki/Perl_(disambiguation))
    URL: http://en.wikipedia.org/wiki/Perl_(disambiguation)

As you can see, the URL encapsulated by brackets are scanned properly. I have not encountered an issue after changing the URL scanner in my Twitter client to this new scanner.

The code can be found in my [GitHub repository](https://github.com/aki-null/URLScanner). The library is licensed under BSD license, so feel free to use it in your own application.
