---
layout: post
title: Testing release builds on iOS
category: articles
tags: [tips,ios]
---

A while ago I was trying to reproduce a bug that some of my users were reporting. I tried every permutation of their descriptions that would trigger the bug every time for them, but I couldn't get it "working" (in other words, "not working") on my side.

Until I realized I was testing on my debug build, while my users were obviously using a release build.

A few minutes later, the bug was reproduced, and perhaps an hour later fixed.

It might be obvious, but the release and debug builds are not identical, and it is a good idea to always test the release version before submitting to the App Store. While you can do this by changing the configuration in the scheme editor back and forth when testing, there is a much more convenient way that let's you do this without messing around with the scheme every time you want to test a release build. It just takes a little bit of tinkering up front.

Depending on your particular setup with regards to app ids the first step is easy, otherwise slightly more involved.

### The simple case
If you use a wildcard app id for your app, like com.myAwesomeCompany.\*, you can just set up Xcode to sign with your developer profile even for the release build. The right profile will be used to re-sign the bundle when you submit anyway. Here's how that would look:

<center>
<img src="/images/posts/code-signing-simple.png">
</center>

That's in the Build Settings tab for the project.

### The less simple case
However, things get a little bit more complicated if you are using a separate app id for each app, which is probably the best way to go anyway. It's required in case you want iCloud support, or use Game Center, or have In-App purchases at some point.

In this case, the app id of your developer profile (which is a wildcard app id, e.g. com.myAwesomeCompany.\*) will be different from the distribution one (e.g. com.myAwesomeCompany.myAwesomeApp). The app id is compiled into the app and won't be changed by the code signing, and there will be a mismatch as a result.

One solution is to create another configuration for release build testing, based on the Release configuration. The only difference being that it's set up to sign with the developer profile instead of the distribution one.

To create a new configuration, start by selecting the project and then the Info tab:

<img src="/images/posts/code-signing-create-configuration.png">

Press the + button to create a new configuration, select "Duplicate Release Configuration". Name it however you want to, we'll be using "RT" in this case, for Release Testing. 

Then switch over to the Build Settings tab again, and set up the code signing to use the developer profile for the new RT line that shows up:

<img src="/images/posts/code-signing-complex.png">

### Making the switch convenient
Now, to be able to easily switch over to a release build for testing, we'll create a new Scheme.

Look in the menu for Product -> Manage Schemes... and select it.

Find the gear icon at the bottom of the sheet that pops up, press it and select Duplicate. Name the new scheme "RT MyAwesomeApp" or similarly.

Then to the important step: select the right configuration for the Run action. If you followed the simple case above, just select the Release configuration. For the less simple case, our new configuration RT is the one to use:

<img src="/images/posts/code-signing-scheme.png">

Now you can just select "RT MyAwesomeApp" in the scheme selector to run your app in release mode, either with the simulator or on a real device. Switching back to the regular scheme will run the app in debug mode as usual.

Hope this helps!
