---
layout: post
title: "Beating Amazon"
date: 2013-11-25 21:04:31 -0500
comments: true
categories: javascript
---

Everytime I see a site that **requires** user interaction, it gives me a chance to practice some hackery.
Case in point: [Amazon black friday deals.](http://www.amazon.com/Black-Friday/b/ref=bf2013_bunk_galaxy?node=384082011)

Amazon wants you to wait on the page counting down like you have nothing better to do.
As a programmer I'd rather make a script to do that for me.

It brought me back a bit as their version of jQuery is 1.6.4. Anyway here's the script, you can
open the javascript console with `Ctrl + Shift + J` then paste the code below in or you can use
the bookmarklet:


<h1>
	<small><small><small>bookmarklet:</small></small></small>
	<a onclick="alert('Drag this to your bookmarks bar and when you\'re on the black friday page click it');return false" href='javascript:(function($){function re__waitForIt__serve($item){var interval=setInterval(function(){var $buyButton=$item.find(".btn");if($buyButton.length){clearInterval(interval);$buyButton[0].click();setTimeout(function(){alert("Item (hopefully) available to buy!");$item.css("outline","1px solid green")},5E3)}},100)}jQuery("body").delegate("li[id]","mouseenter.grabber",function(){var $this=$(this);$this.css("outline","1px solid red");$this.delegate("*","click.grabber",function(){re__waitForIt__serve($this);$this.css("outline", "1px solid blue");$("*").unbind(".grabber");alert("All set to buy item");return false});$this.bind("mouseleave.grabber",function(){$this.css("outline","none")})})})(jQuery);'>Amazon Clicker</a>
</h1>

{% highlight javascript %}
(function($) {
    function re__waitForIt__serve($item) {
        var interval = setInterval(function() {
            var $buyButton = $item.find('.btn');
            if ($buyButton.length) {
                clearInterval(interval);
                $buyButton[0].click();
                setTimeout(function() {
                    alert('Item (hopefully) available to buy!');
					$item.css('outline', '1px solid green');
                }, 5000)
            }
        }, 100);
    }
    jQuery('body').delegate('li[id]', 'mouseenter.grabber', function() {
        var $this = $(this);
        $this.css('outline', '1px solid red');
        $this.delegate('*', 'click.grabber', function() {
            re__waitForIt__serve($this);
            $this.css('outline', '1px solid blue');
            $('*').unbind('.grabber')
            alert('All set to buy item');
            return false;
        });
        $this.bind('mouseleave.grabber', function() { $this.css('outline', 'none'); });
    });
})(jQuery);
{% endhighlight %}

(Tested in Chrome)

Think there's a better way to do this? Did you ever have to do something like this?  
Leave a comment below
