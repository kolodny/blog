---
layout: post
title: "Circular Doubly Linked List"
date: 2013-10-18 11:10
comments: true
categories: javascript algorithm
---

I was working on a cycling plugin for javascript and kept getting bogged down by accessing the prev and next elements in the array. I kept having to do things like:
{% highlight javascript %}
function step() {
	var currentItem = items[currentIndex];
	var nextItem = items[(currentIndex + 1) % items.length];
	var prevItem = items[(items.length + currentIndex - 1) % items.length];
	// more code
	index = (index + 1) % items.length
}
{% endhighlight %}

While this is doable it just seems like a pain.  
Giving it more thought I realized that a [circular doubly linked list](http://en.wikipedia.org/wiki/Linked_list#Circular_list) would be a great solution to this:

<div data-height="257" data-theme-id="1527" data-slug-hash="cGdke" data-user="kolodny" data-default-tab="js" class='codepen'><pre><code>

var list = [{label: &#x27;first&#x27;}, {label: &#x27;second&#x27;}, {label: &#x27;third&#x27;}];
var circled = circleLink(list);

assert(circled[0].next === circled[1]);
assert(circled[1].next === circled[2]);
assert(circled[2].next === circled[0]);

assert(circled[0].prev === circled[2]);
assert(circled[1].prev === circled[0]);
assert(circled[2].prev === circled[1]);

document.write(circled[0].index + &#x27;&lt;br&gt;&#x27;);
document.write(circled[1].index + &#x27;&lt;br&gt;&#x27;);
document.write(circled[2].index + &#x27;&lt;br&gt;&#x27;);
document.write(circled[0].next.next.next.prev.prev.index + &#x27;&lt;br&gt;&#x27;);

function circleLink(array) {
  var linkedListCircle = []
  for (var i = 0; i &lt; array.length; i++) {
    linkedListCircle[i] = array[i];
    linkedListCircle[i].index = i;
    if (i &gt; 0) {
      linkedListCircle[i].prev = linkedListCircle[i - 1];
    }
  }
  linkedListCircle[0].prev = linkedListCircle[i - 1]
  for (var i = 0; i &lt; array.length - 1; i++) {
    linkedListCircle[i].next = linkedListCircle[i + 1];
  }
  linkedListCircle[i].next = linkedListCircle[0];
  return linkedListCircle;
}
function assert(assertion) { document.write((assertion ? true : false) + &#x27;&lt;br&gt;&#x27;) }
</code></pre>
<p>See the Pen <a href='http://codepen.io/kolodny/pen/cGdke'>%= penName %></a> by Moshe Kolodny (<a href='http://codepen.io/kolodny'>@kolodny</a>) on <a href='http://codepen.io'>CodePen</a></p>
</div><script async src="https://codepen.io/assets/embed/ei.js"></script>