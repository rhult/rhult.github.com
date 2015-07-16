---
layout: post
title: Adding a preference to launch sandboxed app on login
category: articles
tags: [code,osx]
---

Woah, time flies! I haven't posted anything here in a long time.

Back in 2009, I was working on Harmony that was later renamed to <a href="http://itunes.apple.com/app/id405918679?mt=12">Break Reminder</a>, and wrote some blog posts about various problems I solved in that app. One example was to add a preference for "launch on login", which used the Shared File List API (see <a href="/articles/preference-to-launch-on-login/">old blog post</a>).

<img src="/images/posts/launch-on-login-break-reminder.png">

 That no longer works in the sandboxed world, but fortunately there is a new way to achieve the same effect.

You need to add a small helper app inside your bundle and tell the system to auto-launch that helper on login. The helper can then start your real app. This is a little bit more complicated than before, since you need to add the helper and make sure it is copied into your app bundle. Let's start with the simpler part first.

The basic idea is still the same as in the old post: to use Cocoa bindings to connect the model to the user interface, so my controller still has the same property:

{% highlight objc %}
@property(nonatomic, assign) BOOL launchOnLogin;
{% endhighlight %}

Adding and removing the helper app to the list of auto-launched apps becomes:

{% highlight objc %}
static NSString const *kLoginHelperBundleIdentifier = <the id of the helper app here>

- (void)addLoginItem
{
    if (!SMLoginItemSetEnabled((CFStringRef)kLoginHelperBundleIdentifier, true)) {
        NSLog(@"SMLoginItemSetEnabled(..., true) failed");
    }
}

- (void)removeLoginItem
{
    if (!SMLoginItemSetEnabled((CFStringRef)kLoginHelperBundleIdentifier, false)) {
      NSLog(@"SMLoginItemSetEnabled(..., false) failed");
    }
}

- (void)setLaunchOnLogin:(BOOL)value
{
    if (!value) {
        [self removeLoginItem];
    } else {
        [self addLoginItem];
    }
}
{% endhighlight %}

Checking whether the app is currently in the list, for the getter implementation:

{% highlight objc %}
- (BOOL)launchOnLogin
{
    NSArray *jobs = (NSArray *)SMCopyAllJobDictionaries(kSMDomainUserLaunchd);
    if (jobs == nil) {
        return NO;
    }

    if ([jobs count] == 0) {
        CFRelease((CFArrayRef)jobs);
        return NO;
    }

    BOOL onDemand = NO;
    for (NSDictionary *job in jobs) {
        if ([kLoginHelperBundleIdentifier isEqualToString:[job objectForKey:@"Label"]]) {
            onDemand = [[job objectForKey:@"OnDemand"] boolValue];
            break;
        }
    }

    CFRelease((CFArrayRef)jobs);
    return onDemand;
}
{% endhighlight %}

So far, things are fairly similar to the old way. Now let's add the helper bundle. This part assumes you are fairly well acquainted with Xcode and Cocoa, so better buckle up!

Add the helper using "File -> New -> Target..." and select a Cocoa Application. Make sure to make the helper sandboxed (enable the entitlement in the Summary pane for the target). The helper shouldn't have any UI, so it's a good idea to set the key LSUIElement to true in the helper's Info.plist, so we don't get an icon in the dock.

Time for the magic that launches the main app. Add this to the helper's app delegate:

{% highlight objc %}
- (void)applicationDidFinishLaunching:(NSNotification *)notification
{
    // Get the path for the main app bundle from the helper bundle path.
    NSString *basePath = [[NSBundle mainBundle] bundlePath];
    NSString *path = [basePath stringByDeletingLastPathComponent];
    path = [path stringByDeletingLastPathComponent];
    path = [path stringByDeletingLastPathComponent];
    path = [path stringByDeletingLastPathComponent];

    // Launch the executable inside the app, seems to work better according to this (and my testing seems to agree):
    // http://stackoverflow.com/questions/9011836/sandboxed-helper-app-can-not-launch-the-correct-parent-application?rq=1
    // But we also fall back to the app in case this is a bug that will get fixed in an OS X update.

    // Note: Replace with the real name of the main app.
    NSString *pathToExecutable = [path stringByAppendingPathComponent:@"Contents/MacOS/My App"];

    if ([[NSWorkspace sharedWorkspace] launchApplication:pathToExecutable]) {
        NSLog(@"Launched executable succcessfully");
    }
    else if ([[NSWorkspace sharedWorkspace] launchApplication:path]) {
        NSLog(@"Launched app succcessfully");
    } else {
        NSLog(@"Failed to launch");
    }

    // We are done, so we might just quit at this point.
    [[NSApplication sharedApplication] terminate:self];
}
{% endhighlight %}

The last step is to copy the helper app into the main bundle. Find the build phases for the main app and add a "Copy Files" phase where you add the helper app. The destination type should be Wrapper and the subpath "Contents/Library/LoginItems":

<img src="/images/posts/launch-on-login-copy-phase.png">

Now, something to keep in mind is that at least with OS X 10.7.x and the early releases of 10.8, the app must be placed in /Applications for this to fully work. That, and the fact that you have to log out and in to test the feature is a bit of a pain.

Hope this helps!
