---
layout: post
title: KVO problems with CALayer on 10.5?
category: articles
tags: [code,osx]
---

I was trying out some code on 10.5 that was originally developed on 10.6, and ran into a number of really strange issues. At first they seemed almost random and unrelated but after debugging for a while it turned out they were all related to key-value observation in some way or another.

After more debugging, it became clear that for some reason I was not getting any observer callbacks and Cocoa bindings weren't working for certain objects.

This looked suspicious, and sure enough, implementing:

{% highlight objc %}
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key
{
    BOOL notifies = [super automaticallyNotifiesObserversForKey:key];
    NSLog(@"Notifies for key %@: %d", key, notifies);
    return notifies;
}
{% endhighlight %}

in one of the affected classes showed that the return value was always <code>NO</code>, even for properties that I had declared myself!

The reason is that CALayer's implementation of <code>automaticallyNotifiesObserversForKey:</code> in 10.5 returns <code>NO</code> for all keys. In 10.6 it works as expected, returning only <code>NO</code> for its own properties.

Fortunately the work-around is simple: just implement the class method yourself and return <code>YES</code> for your own properties:
	
{% highlight objc %}
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key
{
    if ([key isEqualToString:@"myKey"]) {
        return YES;
    }

    return [super automaticallyNotifiesObserversForKey:key];
}
{% endhighlight %}
