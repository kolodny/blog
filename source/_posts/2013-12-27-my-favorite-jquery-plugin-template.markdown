---
layout: post
title: "My Favorite jQuery Plugin Template"
date: 2013-12-27 10:57:16 -0500
comments: true
categories: javascript jquery
---

I've dabbled quite a bit in jQuery and writing plugins for it. I've played around with quite a few different ways to start a plugin, and now I've got a new favorite:

{% highlight javascript %}
;(function($) {
  // multiple plugins can go here
  (function(pluginName) {
    var defaults = {
      color: 'black',
      testFor: function(div) {
        return true;
      }
    };
    $.fn[pluginName] = function(options) {
      options = $.extend(true, {}, defaults, options);
            
      return this.each(function() {
        var elem = this,
          $elem = $(elem);

        // heres the guts of the plugin
          if (options.testFor(elem)) {
            $elem.css({
              borderWidth: 1,
              borderStyle: 'solid',
              borderColor: options.color
            });
          }
      });
    };
    $.fn[pluginName].defaults = defaults;
  })('borderize');
})(jQuery);
{% endhighlight %}

Now let's see how we would use it.
{% highlight javascript %}
$('div').borderize();
$('div').borderize({color: 'red'});
{% endhighlight %}

<iframe width="100%" height="300" src="http://jsfiddle.net/EVL22/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

 Here's some of the reasons that I like this technique

 1. You can still use a default option inside of a override (similar to calling a parent property in class based programming)
 2. Easily change the name of the plugin as long as we use pluginName all over (also there's an insignificant minification advantage of that).
 3. Cleaner (at least in my opinion)

 Point #1 is huge, let's see an example that. Let's say we want to override the `testFor` function but still want the option of defaulting to the original behaviour

{% highlight javascript %}
$('.borderize').borderize({
	testFor: function(elem) {
		var $elem = $(elem);
		if (elem.is('.inactive')) {
			return false;
		} else {
			// calling "parent" function
			return $.fn.borderize.defaults.testFor.apply(this, arguments);
		}
	}
});

{% endhighlight %}

We can even do this with regular properties like this
{% highlight javascript %}
var someVarThatMayBeSet = false;
/* code ... */

$('.borderize').borderize({
	color: someVarThatMayBeSet ? 'red' : $.fn.borderize.defaults.color
});
{% endhighlight %}

<iframe width="100%" height="300" src="http://jsfiddle.net/GDqrC/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Do you have a different style that you like? Leave a comment below

Edit I've changed the `$.each` call to `$.extend(true, {}, defaults, options);` based on [phlyingpenguin](https://news.ycombinator.com/item?id=6971361) comment.