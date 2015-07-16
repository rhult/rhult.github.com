---
layout: post
title: Some quick tips for Inkscape on OS X
category: articles
tags: [tips,osx]
---

I use <a href="http://inkscape.org/" target="_blank">Inkscape</a> for most graphics tasks like drawing icons and artwork for apps. It's a fairly competent application but takes a while to get used to since it's not a native OS X app but instead relies on X11. So to use it on OS X 10.8 or later you need to first install <a href="http://xquartz.macosforge.org/landing/" target="_blank">XQuartz</a> which is the X11 implementation for OS X.

Then there are a few preferences you can tweak in X11 to make it play better with Inkscape.

First, to be able to use the &#8997; key as a modifier you need to enable "Option keys send Alt_L and Alt_R" in the Input section[^1]:

<img src="/images/posts/quick-inkscape-tips-input.png">

Second, to be able to copy and paste vector graphics between Inkscape windows[^2], you need to disable "Update Pasteboard when CLIPBOARD changes" in the Pasteboard section:

<img src="/images/posts/quick-inkscape-tips-pasteboard.png">

In previous releases of Xquartz, you needed to make sure to open Inkscape windows from the same instance (using the File -> Open menu item) to be able to copy and paste between windows, but that seems to be fixed now.

Now for the third and most important part: you just need to beat your head against the wall repeatedly until your muscle memory 
gets used to the oddities of X11 apps under OS X and before you know it you will automatically:

* Use Ctrl instead of &#8984; for key shortcuts like Ctrl-C/V for copy and paste
* Learn to press S very often to activate the Select tool to avoid unintentional changes
* Rely heavily on Undo to undo the many unintentional edits you do in spite of the above
* Be careful not to scroll over sliders since that changes the slider instead of scrolling
* Learn to click already selected objects to move key focus out of text fields like RGBA color fields
* And so on...

This list is mostly tounge-in-cheek, I love Inkscape - it's a very capable tool, it's free and uses a free and open file format, SVG. Another really nice feature is that it is possible to things like exporting to PNG using a command-line interface, so things like automatically exporting icons and images can be done easily. But it does take a little bit of struggling to get over the differences to native OS X apps.

Hope this helps!

[^1]: For example for &#8997;-clicking a stack of objects to select objects that are below the top one.
[^2]: Without this setting, images are transferred as bitmaps instead of vectors.
