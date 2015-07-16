---
layout: post
title: Registering defaults for NSUserDefaults using a property list
category: articles
tags: [code,ios,osx]
---

In a previous post about <a href="/articles/using-property-lists/">using property lists</a>, I wrote a little about property lists and a use case I had for them. Another one I ran in to recently is related to user defaults:

NSUserDefaults is the system in Mac OS X that handles user preferences. Applications usually register default values at launch time, so that all preferences have a sane default value in case the user hasn't set one for a particular preference. Most examples I've found on the subject do something along the lines of:

{% highlight objc %}
// Register user defaults in the class initializer.
+ (void)initialize
{
    NSDictionary *appDefaults = [NSDictionary dictionaryWithObjectsAndKeys:
                                 @"YES", @"ShowToolBar",
                                 @"NO", @"AutoSaveEnabled",
                                 // ... lots of objects and keys here,
                                 nil];

    [[NSUserDefaults standardUserDefaults] registerDefaults:appDefaults];
}
{% endhighlight %}

However, if you have many keys, or just want to make it possible to change them without recompiling, they can be managed through a property list file instead of doing it programmatically. To do that, first create a property list by selecting File → New File... or pressing ⌘-N in Xcode and selecting Property List in the Other category. Then add the default values using the property list editor.

Assuming the plist file is named UserDefaults.plist, the code can then be changed to:

{% highlight objc %}
+ (void)initialize
{
    NSString *defaultsPath = [[NSBundle mainBundle] pathForResource:@"UserDefaults"
                                                             ofType:@"plist"];
    NSDictionary *appDefaults = [NSDictionary dictionaryWithContentsOfFile:defaultsPath];

    [[NSUserDefaults standardUserDefaults] registerDefaults:appDefaults];
}
{% endhighlight %}
