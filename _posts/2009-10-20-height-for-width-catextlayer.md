---
layout: post
title: Height-for-width layout with CATextLayer
category: articles
tags: [code,osx]
---

For a project I'm working on, I needed a Core Animation text layer that could adapt its height depending on the available width. This is commonly called a "height-for-width" layout. The stock CATextLayer and CAConstraintLayoutManager can't really do this, but since you can implement custom layout managers, that's where I started.

The two most important methods of the layout manager protocol are:

{% highlight objc %}

// To implement in the CALayoutManager implementation:
- (CGSize)preferredSizeOfLayer:(CALayer *)layer;
- (void)layoutSublayersOfLayer:(CALayer *)layer;

{% endhighlight %}

The idea was to override <code>preferredSizeOfLayer:</code> to special-case any CATextLayers that had wrapping enabled and calculate the needed height for any given width. My first somewhat naive attempt was to use the AppKit's NSString additions like <code>sizeWithAttributes:</code> or <code>boundingRectWithSize:options:attributes:</code>. The results were almost right but not quite the same as what Core Animation itself would get. Using this approach, the layer would adjust its height according to the available width, but not exactly right. For some widths, the height would be one line to tall or short.

The second option was to use the Cocoa text system and put together the various pieces in order to measure the height with some more control over the process. This consisted of creating an NSTextStorage instance (and setting up the text and its attributes), an NSTextContainer instance (with the right width, and "infinite" height), and an NSLayoutManager instance. After putting those together and forcing a layout which is otherwise done lazily, I got the bounds from the layout manager. The code looked something like this:

{% highlight objc %}

// Measures the height needed for a given width using the Cocoa text system:
- (CGSize)frameSizeForTextLayer:(CATextLayer *)layer
{
    NSTextStorage *storage;
    if ([layer.string isKindOfClass:[NSAttributedString class]]) {
        storage = [[NSTextStorage alloc] initWithAttributedString:layer.string];
    } else {
        storage = [[NSTextStorage alloc] initWithString:layer.string];

        /* ... set up the attributes for the storage, like the font ... */
    }

    NSTextContainer *container = [[NSTextContainer alloc] initWithContainerSize:NSMakeSize(layer.bounds.size.width, FLT_MAX)];
    NSLayoutManager *manager = [[NSLayoutManager alloc] init];

    [manager addTextContainer:container];
    [storage addLayoutManager:manager];

    // The text layer doesn't use fragment line padding.
    [container setLineFragmentPadding:0];

    // Force layout, since it's done lazily.
    [manager glyphRangeForTextContainer:container];

    return [manager usedRectForTextContainer:container].size;
}
{% endhighlight %}


The result was much better this time, as a matter of fact so good that I thought it was finally right. But then I discovered that for some fonts there was still a small difference between CATextLayer's measurements and mine. I could not find any parameters that would remove the differences, but some investigations seemed to indicate that the difference was in how the line heights, line spacing or maximum line height was set up. Or perhaps there is some rounding going on in the Cocoa text system to get text lines to end up on evenly aligned pixel boundaries? Either way, I didn't really feel like going the trial-and-error way to get the (hopefully) right results...

After doing some more debugging in Xcode, it looked like CATextLayer actually uses Core Text directly, so I decided to try that next. This was quite similar to using the Cocoa text system, not surprising as the latter is built as a quite thin layer on top of the former.

Finally, it looked like the results were matching CATextLayer! :) The code that does the text measuring now looked like the following:

{% highlight objc %}

// Measures the height needed for a given width using Core Text:
- (CGSize)frameSizeForTextLayer:(CATextLayer *)layer
{
    NSAttributedString *string = [self attributedStringForTextLayer:layer];
    CTTypesetterRef typesetter = CTTypesetterCreateWithAttributedString((CFAttributedStringRef)string);
    CGFloat width = layer.bounds.size.width;
    
    CFIndex offset = 0, length;
    CGFloat y = 0;
    do {
        length = CTTypesetterSuggestLineBreak(typesetter, offset, width);
        CTLineRef line = CTTypesetterCreateLine(typesetter, CFRangeMake(offset, length));
        
        CGFloat ascent, descent, leading;
        CTLineGetTypographicBounds(line, &ascent, &descent, &leading);
        
        CFRelease(line);
        
        offset += length;
        y += ascent + descent + leading;
    } while (offset < [string length]);
    
    CFRelease(typesetter);
    
    return CGSizeMake(width, ceil(y));
}

{% endhighlight %}

The method <code>attributedStringForTextLayer:</code> sets up an NSAttributedString correctly from the string, font and fontSize properties of a text layer.

At this point, the height-for-width layout was halfway through complete. The remaining issue left to solve was how make the layout manager do something useful with the calculated height and use it as an input for the regular constraints, for example to place something above the text layer, or to make a nice looking box sized to fit the wrapped text. But that, and the full source code will be available in an upcoming post soon. Stay tuned!
