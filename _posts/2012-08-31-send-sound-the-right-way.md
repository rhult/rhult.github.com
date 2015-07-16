---
layout: post
title: Send system sounds the right way (not over AirPlay)
category: articles
tags: [code,osx]
---

This is another little bit of code that I needed for my Mac app <a href="http://itunes.apple.com/app/id405918679?mt=12">Break Reminder</a>. When it's time for a break, Break Reminder can optionally play a sound to get your attention.

<img src="/images/posts/send-sound-the-right-way.png">

I use the NSSound API to play a simple chime-style sound, which can look something like:

{% highlight objc %}
    [[NSSound soundNamed:@"Chime"] play];
{% endhighlight %}

Starting with OS X 10.8 (Mountain Lion), the user can choose to set an AirPlay device as sound output, which can be something like an AirPort Express connected to a stereo. This is great for music and the like, but for chimes, not so great! 

Fortunately, there is a pretty simple solution. NSSound has a method to set the playback device to use:

{% highlight objc %}
    NSSound *sound = [NSSound soundNamed:@"Chime"];

    NSString *deviceID = ...
    [sound setPlaybackDeviceIdentifier:deviceID];

    [sound play];
{% endhighlight %}

We just need to get the device for system-kind-of-sounds, like the built-in chimes and boings and whatnots. We can use CoreAudio to get the identifier for that:

{% highlight objc %}
- (NSString *)systemAudioDeviceID
{
    AudioDeviceID systemOutputDevice = 0;
    UInt32 size = sizeof(AudioDeviceID);
    AudioObjectPropertyAddress address = {
        kAudioHardwarePropertyDefaultSystemOutputDevice,
        kAudioObjectPropertyScopeGlobal,
        kAudioObjectPropertyElementMaster
    };
    OSStatus err = AudioObjectGetPropertyData(kAudioObjectSystemObject, &address, 0, NULL, &size, &systemOutputDevice);
    if (err != noErr) {
        ... handle error
        return nil;
    }

    CFStringRef deviceUID = NULL;
    size = sizeof(deviceUID);
    AudioObjectPropertyAddress uidAddress = {
        kAudioDevicePropertyDeviceUID,
        kAudioDevicePropertyScopeOutput,
        /*channel*/ 0
    };
    err = AudioObjectGetPropertyData(systemOutputDevice, &uidAddress, 0, NULL, &size, &deviceUID);
    if (err != noErr) {
        ... handle error
        return nil;
    }

    return (__bridge_transfer NSString *)deviceUID;
}
{% endhighlight %}


Just plug the identifier into the NSSound and you should be good to go.

Hope this helps!
