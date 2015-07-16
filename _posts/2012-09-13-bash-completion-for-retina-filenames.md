---
layout: post
title: Bash tab completion for retina filenames
category: articles
tags: [tips,osx]
---

When doing iOS and OS X development you often need some files with "@" in the filename for retina images. If you're anything like me and do some work in the terminal involving those files, you have probably noticed that the shell (bash in this case) treats anything with the "@" character in it like a hostname. Any tab completion you do will then try to construct various hostnames based on what you typed. Annoying!

<img src="/images/posts/retina-completion-gone-bad.png">

If you don't need the hostname completion, you can disable it, and instead get what you expect when you press tab for those filenames. Put the following in your ~/.profile file:

{% highlight objc %}
shopt -u hostcomplete
{% endhighlight %}

<br>
That will get you want you want:

<img src="/images/posts/retina-completion-working.png">

Hope this helps!
