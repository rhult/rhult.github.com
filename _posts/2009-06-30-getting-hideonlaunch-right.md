---
layout: post
title: Getting HideOnLaunch right
category: articles
tags: [code,osx]
---

In a previous post I described how to set up an app to <a href="/articles/preference-to-launch-on-login/">launch automatically on login</a>. After posting that, I've noticed that it seems like the "Hide on launch" property often seems to be mixed up with the <code>kLSSharedFileListItemHidden</code> key (for an example, see <a href="http://lists.apple.com/archives/Cocoa-dev/2008/Feb/msg00632.html">this thread</a> from 2008 on Cocoa-dev).

So in an attempt to improve chances of people finding the right answer when searching for the incorrect key <code>kLSSharedFileListItemHidden</code>, here is the relevant snippet again:

{% highlight objc %}
NSDictionary *properties =
    [NSDictionary dictionaryWithObject:[NSNumber numberWithBool:YES]
                                // NOTE, notice the key name:
                                forKey:@"com.apple.loginitem.HideOnLaunch"];

LSSharedFileListItemRef itemRef = 
    LSSharedFileListInsertItemURL(loginItemsListRef,
                                  kLSSharedFileListItemLast,
                                  NULL,
                                  NULL,
                                  (CFURLRef)bundleURL,
                                  (CFDictionaryRef)properties,
                                  NULL);
...
{% endhighlight %}
