---
layout: post
title: "When Programming Goes Bad - Debugging The Impossible"
date: 2014-07-21 10:28:09 -0400
comments: true
categories: javascript
---

Sometimes code for a project gets so big and so deeply indirected that debugging is nearly impossible.

We had a bug similar to this where a controller was changing models which triggered avalanches of cascades that no human could follow. The only thing we knew was that a select with an id of `elusive` was being changed through jQuery.

After trying in vain to figure out where the change was happening, I had an [aha! moment](http://en.wikipedia.org/wiki/Eureka_effect)

This is basically how I solved it:


{% highlight javascript %}
(function($) {
	var oldAttr = $.fn.attr;
	var oldProp = $.fn.prop;
	$.fn.attr = function() {
		if (this.id === 'elusive') {
			debugger;			
		}
		return oldAttr.apply(this, arguments);
	};
	$.fn.prop = function() {
		if (this.id === 'elusive') {
			debugger;			
		}
		return oldProp(this, arguments);
	};
})(jQuery);
{% endhighlight %}

Then it's just a matter of looking at the call stack and working backwards from there.