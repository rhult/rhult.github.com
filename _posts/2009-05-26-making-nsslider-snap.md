---
layout: post
title: Making an NSSlider snap to tick marks
category: articles
tags: [code,osx]
---

Have you noticed how the sliders in the energy savings panel in the system preferences behave a little differently from your average slider? They have this nice touch of resistance when you drag the slider across a tick mark, making it easy to end up exactly on a mark. I wanted the same behavior for a slider in my app, and found out that it isn't builtin in the slider. The way I solved it was by connecting an action to the slider, where I manually restrict the value a bit:

{% highlight objc %}
- (IBAction)sliderValueChanged:(id)sender
{
    double range = [sender maxValue] - [sender minValue];
    double tickInterval = range / ([sender numberOfTickMarks] - 1);    

    double relativeValue = [sender doubleValue] - [sender minValue];

    // Get the distance to the nearest tick.
    int nearestTick = round(relativeValue / tickInterval);
    double distance = relativeValue - nearestTick * tickInterval;

    // Change the check here depending on how much resistance you
    // want, or if you don't want it to depend on the tick interval.
    if (fabs(distance) < tickInterval / 8) {
        [sender setDoubleValue:[sender doubleValue] - distance];
    }
}
{% endhighlight %}

Then you have to make the slider continously perform the action when moved, instead of just when releasing it. This can be done either by checking the "Continous" check button in Interface Builder or programmatically using:

{% highlight objc %}
- (void)setContinuous:(BOOL)flag
{% endhighlight %}

on the slider instance.

There might be a better way to do this, if anyone knows about it I'm all ears :)
