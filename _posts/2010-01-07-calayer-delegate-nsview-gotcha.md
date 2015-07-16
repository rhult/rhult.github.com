---
layout: post
title: Small gotcha when using NSView as delegate for CALayer
category: articles
tags: [code,osx]
---

I ran into another small unexpected behavior when working on some Core Animation code recently. I had a subclass of NSView that acted as a layer-hosting view. The layers that the view hosted had the view set as delegate to do some simple drawing.

I didn't want the layers to animate their bounds so I did the usual:

{% highlight objc %}
[layer setActions:[NSDictionary dictionaryWithObjectsAndKeys:
                   [NSNull null], @"bounds",
                   [NSNull null], @"position",
                   nil]];
{% endhighlight %}

...which to my surprise didn't work. After digging around for a while I found out that NSView implements <code>actionForLayer:forKey</code>, which has precedence over the actions dictionary. The quick and dirty solution was to override it and return a <code>null</code> action for the relevant keys:

{% highlight objc %}
- (id<CAAction>)actionForLayer:(CALayer *)layer forKey:(NSString *)event
{
    if ([event isEqualToString:@"position"] || [event isEqualToString:@"bounds"]) {
        return (id<CAAction>)[NSNull null];
    }

    return nil;
}
{% endhighlight %}

However, my guess is that NSView is not really meant to be used as delegate for its hosted layers like this. A better design in most cases would be to either subclass the layer to do its own drawing, or to use a separate controller for event handling or other book keeping for the layer(s).

So in reality this is not really a problem but I thought I'd post it here in case someone else runs into this problem.
