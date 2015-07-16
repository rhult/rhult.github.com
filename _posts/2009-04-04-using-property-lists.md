---
layout: post
title: Using property lists
category: articles
tags: [property list,code,ios,osx]
---

Before entering the Cocoa world, whenever I needed to store and retrieve some small amount of data in an application, I usually handcrafted an XML format and wrote a matching XML parser using either DOM or SAX. This often meant a lot of uninspiring code duplication. On Mac OS X, there is a standardized format that is used throughout the system called property list or "plist". Not only are there APIs to parse plists into the familiar data structures in Cocoa, but there is also a builtin editor for plists in Xcode.

This came in handy recently in some code I was working on. I needed to store 20 or 30 pairs of strings and select one pair randomly from time to time. Instead of entering the strings in the code, they were put into a plist file.

##Create and edit a plist
It's easy to create a property list. In Xcode, just select File → New File... or press cmd-N, and select the template Property List in the Other category. Then add the data you wish to, using the property list editor in Xcode. You can of course also handwrite the XML using any text editor.

##Use the data
Assume that the property list is structured with a top-level key called Pairs, whose value is an array of our string pairs. Each pair in turn is also an array, with two strings in each. The code to read the list could then look as follows:

{% highlight objc %}
- (void)readStringPairs
{
    NSString *path = [[NSBundle mainBundle] pathForResource:@"MyFileName"
                                                     ofType:@"plist"];
    NSDictionary *toplevelDict = [NSDictionary dictionaryWithContentsOfFile:path];

    // Get the array of pairs, retain it as we need it later.
    pairs = [[toplevelDict valueForKey:@"Pairs"] retain];
}

- (void)randomizeStrings
{
    // Get a random pair, represented by an array.
    NSArray *pair = [pairs objectAtIndex:arc4random() % [pairs count]];

    // Get the two strings.
    NSString *name = [pair objectAtIndex:0];
    NSString *description = [pair objectAtIndex:1];

    // Do something with the strings here.
}
{% endhighlight %}

If you need something a little bit more flexible or complex, you can nest dictionaries and arrays in the plist as well. As a matter of fact, my original code doesn't only have two strings per pair, but one string and an array of strings.
