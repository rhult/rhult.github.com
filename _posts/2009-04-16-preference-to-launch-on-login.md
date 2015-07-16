---
layout: post
title: Adding a preference to launch app on login using the Shared File List API
category: articles
tags: [code,osx]
---

<i>Note: See new post for doing this in a <a href="/articles/sandboxed-launch-on-login/">sandboxed app</a>.</i>

I wanted to make my app launch automatically on login, and looked around for ways to do that programmatically. I first found two ways to do it, both a little bit less than ideal: either using Apple Events to talk to System Events, or by using NSUserDefaults (or CFPreferences) to tweak values in the persistent domain <code>loginwindow</code> (as opposed to the app's own persistent domain where the preferences for the app are stored). The Apple Events way seemed a bit hackish to me, as did the NSUserDefaults way, considering it meant poking at preferences you don't own.

Nonetheless, I decided to try the latter, so I went ahead and implemented a simple controller that exposed only one property that I could then bind to a check box button in Interface Builder using Cocoa bindings:

{% highlight objc %}
@property BOOL launchOnLogin;
{% endhighlight %}

To show more clearly what I did, here's the part that reads the current login items:

{% highlight objc %}
NSUserDefaults *defaults;
NSDictionary *domain;

defaults = [NSUserDefaults standardUserDefaults];
  
domain = [defaults persistentDomainForName:@"loginwindow"];
if (domain) {
    NSArray *items = [domain objectForKey:@"AutoLaunchedApplicationDictionary"];

    for (NSDictionary *entry in items) {
        NSString *entryPath = [entry objectForKey:@"Path"];
        // ... do something with the entry ...
    }
}
{% endhighlight %}

The writing part is similar, getting a copy of the existing array, then adding or removing our own item and then setting as the new array:

{% highlight objc %}
[domain setObject:updatedItems forKey:@"AutoLaunchedApplicationDictionary"];

// Update the setting.
[defaults setPersistentDomain:domain forName:@"loginwindow"];

// Write changes to disk so the system settings will be up to date
// if opened before the changes are automatically synced.
[defaults synchronize];
{% endhighlight %}

While this approach works in general, there's a problem: the login items list in the system settings panel is not updated when changing the setting in my app's preferences, and the other way around. Maybe not a big problem, but it made me search some more, which led me to the following note in the release notes for the Launch Services framework in Leopard:

> The Shared File List API is new to Launch Services in Mac OS X Leopard. This API provides access to several kinds of system-global and per-user persistent lists of file system objects, such as recent documents and applications, favorites, and login items. For details, see the new interface file LSSharedFileList.h.

The mentioned header file is the only documentation for this new API, but it is quite straightforward: you create an instance of the right list, in this case the login items list for the session. Then there is a bunch of things you can do with the list, what's interesting for our use case is to get a copy of the array in the list, to add or remove an item, or to register an observer callback for a list. The callback is called when the list changes, which makes it possible for the UI to update accordingly.

Since my app already depends on Leopard, I decided to rewrite my code using LSSharedFileList. The UI was already set up to use the simple controller described above, so a quick rewrite of the controller was enough. I create the list in my init method and add an observer:

{% highlight objc %}
loginItemsListRef = LSSharedFileListCreate(NULL,
                                           kLSSharedFileListSessionLoginItems,
                                           NULL);
if (loginItemsListRef) {
    // Add an observer so we can update the UI if changed externally.
    LSSharedFileListAddObserver(loginItemsListRef,
                                CFRunLoopGetMain(),
                                kCFRunLoopCommonModes,
                                loginItemsChanged,
                                self);
}
{% endhighlight %}

The matching tearing down is done in dealloc:

{% highlight objc %}
if (loginItemsListRef) {
    LSSharedFileListRemoveObserver(loginItemsListRef,
                                   CFRunLoopGetMain(),
                                   kCFRunLoopCommonModes,
                                   loginItemsChanged,
                                   self);
    CFRelease(loginItemsListRef);
}
{% endhighlight %}

The callback for the observer is a plain old C funtion:

{% highlight objc %}
static void
loginItemsChanged(LSSharedFileListRef listRef, void *context)
{
    LoginItemsController *controller = context;

    // Emit change notification for the bindnings. We can't do will/did
    // around the change but this will have to do.
    [controller willChangeValueForKey:@"launchOnLogin"];
    [controller didChangeValueForKey:@"launchOnLogin"];
}
{% endhighlight %}

The rest of the code just needs to check if our app bundle is listed in the login items list in the getter for the property, and add it or remove it in the setter:

{% highlight objc %}
// Get an NSArray with the items.
- (NSArray *)loginItems
{
    CFArrayRef snapshotRef = LSSharedFileListCopySnapshot(loginItemsListRef, NULL);
    
    // Use toll-free bridging to get an NSArray with nicer API
    // and memory management.
    return [NSMakeCollectable(snapshotRef) autorelease];
}

// Return a CFRetained item for the app's bundle, if there is one.
- (LSSharedFileListItemRef)mainBundleLoginItemCopy
{
    NSArray *loginItems = [self loginItems];
    NSURL *bundleURL = [NSURL fileURLWithPath:[[NSBundle mainBundle] bundlePath]];
    
    for (id item in loginItems) {
        LSSharedFileListItemRef itemRef = (LSSharedFileListItemRef)item;
        CFURLRef itemURLRef;
        
        if (LSSharedFileListItemResolve(itemRef, 0, &itemURLRef, NULL) == noErr) {
            // Again, use toll-free bridging.
            NSURL *itemURL = (NSURL *)[NSMakeCollectable(itemURLRef) autorelease];
            if ([itemURL isEqual:bundleURL]) {
                CFRetain(item);
                return (LSSharedFileListItemRef)item;
            }
        }
    }
    
    return NULL;
}

- (void)addMainBundleToLoginItems
{
    // We use the URL to the app itself (i.e. the main bundle).
    NSURL *bundleURL = [NSURL fileURLWithPath:[[NSBundle mainBundle] bundlePath]];
    
    // Ask to be hidden on launch. The key name to use was a bit hard to find, but can
    // be found by inspecting the plist ~/Library/Preferences/com.apple.loginwindow.plist
    // and looking at some existing entries. Thanks to Anders for the hint!
    NSDictionary *properties;
    properties = [NSDictionary dictionaryWithObject:[NSNumber numberWithBool:YES]
                                             forKey:@"com.apple.loginitem.HideOnLaunch"];
    
    LSSharedFileListItemRef itemRef;
    itemRef = LSSharedFileListInsertItemURL(loginItemsListRef,
                                            kLSSharedFileListItemLast,
                                            NULL,
                                            NULL,
                                            (CFURLRef)bundleURL,
                                            (CFDictionaryRef)properties,
                                            NULL);
    if (itemRef) {
        CFRelease(itemRef);
    }
}

- (void)removeMainBundleFromLoginItems
{
    // Try to get the item corresponding to the main bundle URL.
    LSSharedFileListItemRef itemRef = [self mainBundleLoginItemCopy];
    if (!itemRef)
        return;
    
    LSSharedFileListItemRemove(loginItemsListRef, itemRef);
    
    CFRelease(itemRef);
}

#pragma mark Property accessor methods
- (BOOL)launchOnLogin
{
    if (!loginItemsListRef)
        return NO;

    LSSharedFileListItemRef itemRef = [self mainBundleLoginItemCopy];    
    if (!itemRef)
        return NO;
    
    CFRelease(itemRef);
    return YES;
}

- (void)setLaunchOnLogin:(BOOL)value
{
    if (!loginItemsListRef)
        return;
    
    if (!value) {
        [self removeMainBundleFromLoginItems];
    } else {
        [self addMainBundleToLoginItems];
    }
}
{% endhighlight %}

That's all! Not only does this give you synchronization between your preferences panel and the system settings, but also feels more correct than poking at a different application's or subsystem's persistent domain.

Note: There is also a "seed" for each list that can be used to check if the list has changed. Using that you can make sure that any change notification is not emitted unless necessary (see <code>LSSharedFileListGetSeedValue()</code>). The example above doesn't do that, so there will be notification sent out when toggling the setting from the app's preferences.
