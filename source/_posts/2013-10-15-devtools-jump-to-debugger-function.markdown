---
layout: post
title: "Devtools - Jump To Debugger"
date: 2013-10-15 14:49
comments: true
categories: javascript dev-tools
---
Here's a quick trick to jump into a breakpoint without having to hunt through the sources for the function.
Assume you have something like this ![Assume something is not working with this](http://i.imgur.com/6E4a1lh.png)

It would be nice if we could just jump to the function with a breakpoint toggled. Here's one way to do that:
Type in `> (function() { debugger; theFunctionToStepThrough(); })()`
![Type it like this](http://i.imgur.com/iCIt3Zj.png)
Now we get to this screen
![debugger caught in vm](http://i.imgur.com/wzEJLcO.png)
And if you hit F11 or click the step into button we get to this:
![Done!](http://i.imgur.com/W440Zai.png)
Hope it comes in handy

---

It seems that you don't have to even wrap it in an [IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/), instead all you need to do is:
{% highlight javascript %}
> debugger; test();
{% endhighlight %}