---
layout: post
title: NSSound and AirPlay
category: articles
tags: [code,osx]
---

In a <a href="/articles/send-sound-the-right-way/">previous post</a>, I described how to set up an NSSound to be played over the system device instead of the default output (which could be an AirPlay device).

I use this in my Mac app <a href="http://itunes.apple.com/app/id405918679?mt=12">Break Reminder</a>, to make sure the notification sound doesn't play over AirPlay. After running Break Reminder for a while after fixing this, I noticed that sometimes the sound would start playing over AirPlay anyway! What gives?

This is what I did in the previous post:

{% highlight objc %}
NSSound *sound = [NSSound soundNamed:@"Chime"];

NSString *deviceID = ...
[sound setPlaybackDeviceIdentifier:deviceID];

[sound play];
{% endhighlight %}

As far as I can see, there is a bug in NSSound that prevents the playback device setting to work, seemingly after a long time (probably days rather than hours). Since Break Reminder runs all the time, this bug is bound to pop up.

Some thinking and guesswork led me to try to allocate the sound using 

{% highlight objc %}
- (id)initWithContentsOfFile:(NSString *)filepath byReference:(BOOL)byRef
{% endhighlight %}

which seems to work better. My guess is that since this will explicitly load the sound each time it is needed, the playback device will always be set, working around a possible bug in the cache used by soundNamed:(?).

If someone else runs into this, please do what I did: file a bug using <a href="https://bugreport.apple.com">Apple's bug report tool</a> (I got #12506583).

Hope this helps!
