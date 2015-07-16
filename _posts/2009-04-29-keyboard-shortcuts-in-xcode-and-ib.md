---
layout: post
title: Keyboard shortcuts in Xcode and Interface Builder
category: articles
tags: [tips,osx]
---

If you, like me, are trying to avoid using the mouse (for ergonomic reasons) as much as possible, you probably already have noticed that the Mac is quite alright when it comes to keyboard accessibility. This includes Xcode and Interface Builder, even though the latter by nature requires quite a bit of mouse wrestling. There are also some nice features that can help you having to type less.

Recently, I have been trying to collect the most useful keyboard shortcuts and really learn them so I'm not tempted to use the mouse more than necessary. Here's the list so far:

### Xcode
Besides all the normal text editing shortcuts, I often use those: (⇧ = shift, ⌘ = cmd, ^ = ctrl, ⌥ = option)

- ⇧⌘E = Zoom the editor by hiding the list above it
- ⇧⌘D = Open quickly, useful for quickly open any file in the project our elsewhere
- ^⌘T = Edit all in scope, this saves a lot of tedious editing
- ^/ = Next placeholder in completions
- ^. = Toggle between completions
- ⌥⌘-Up = Toggle between the header/source
- ^1 = Pop up the file history menu, useful when navigating the project files

The shortcut ^. deserves some extra attention, as it also completes text macros which can save a lot of typing. There is a whole slew of macros that you can use, just a few examples:

- pim = expands to #import "file", highlighting file so you can change it easily
- a = expands to the standard alloc/init combination
- init = expands to a standard init skeleton
- dealloc = expands to the standard dealloc skeleton

### Interface Builder
Interface builder also has a few ones I often use:

- ⌘ while resizing a window = live autoresizing
- ^⌘ + up/down = select parent/child of selected view
- ^⌘ + left/right = select previous/next sibling

### Global shortcuts
Finally I have a tiny list of desktop wide shortcuts (obviously in addition to the well known ones like ⌘-Tab etc) I sometimes find useful:

- ^F2 = Focus the application menu

... I said it's tiny!

I hope this can be useful for others as well.
