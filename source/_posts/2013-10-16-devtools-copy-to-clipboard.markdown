---
layout: post
title: "Devtools Copy To Clipboard"
date: 2013-10-16 11:08
comments: true
categories: javascript dev-tools
---
I was playing around with devtools and i saw in the autocomplete list of functions had `copy()`  
It does what it sounds like it does (copies text to the clipboard)
{% highlight javascript %}
> copy("I'm about to hit ctrl+v")
undefined
> I'm about to hit ctrl+v
{% endhighlight %}
Very useful for when you need to manipulate large chunks of text.

Works in Devtools and Firebug (at least)