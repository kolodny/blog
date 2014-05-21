---
layout: post
title: "Object.observe nested objects"
date: 2014-05-21 11:20:31 -0400
comments: true
categories: javascript
---

A great article came out yesterday: [Data-binding Revolutions with Object.observe()](http://www.html5rocks.com/en/tutorials/es7/observe/)

If you didn't read it yet, go read it then come back here.

Ok the basic idea of that article is that you can do things like this



{% highlight javascript %}
var model = {};

// Which we then observe
Object.observe(model, function(changes){

    // This asynchronous callback runs and aggregates changes
    changes.forEach(function(change) {

        // Letting us know what changed
        console.log(change.type, change.name, change.oldValue);
    });

});
model.adding = 'a prop';
model.nested = { obj: 'thing' };
model.nested.obj = 'another thing';
{% endhighlight %}

running that in the console we get:

{% highlight javascript %}
add adding undefined
add nested undefined 
{% endhighlight %}

Now it would be cool if there was a way to observe nested objects. Here's a simple function to do so:

{% highlight javascript %}
function observeNested(obj, callback) {
    Object.observe(obj, function(changes){
        changes.forEach(function(change) {
            if (typeof obj[change.name] == 'object') {
                observeNested(obj[change.name], callback);
            }
        });
        callback.apply(this, arguments);
    });
}
{% endhighlight %}

Now at first glance it would seem this doesn't work:

{% highlight javascript %}
function observeNested(obj, callback) {
    Object.observe(obj, function(changes){
        changes.forEach(function(change) {
            if (typeof obj[change.name] == 'object') {
                observeNested(obj[change.name], callback);
            }
        });
        callback.apply(this, arguments);
    });
}

var model = {};

// Which we then observe
observeNested(model, function(changes){
    changes.forEach(function(change) {
        console.log(change.type, change.name, change.oldValue);
    });
});
model.adding = 'a prop';
model.nested = { obj: 'thing' };
model.nested.obj = 'another thing';

{% endhighlight %}
 
Still shows:

{% highlight javascript %}
add adding undefined
add nested undefined 
{% endhighlight %}

But then I remembered that (and I'm quiting from the article):
`Object.observe(), part of a future ECMAScript standard, is a method for asynchronously observing changes to JavaScript objects`

So in the case above since the two assignments to `model.nested` happen right after another, it only logs it as one event, If you were using promises to some other async mechanism it would work as advertised:



{% highlight javascript %}
function observeNested(obj, callback) {
    Object.observe(obj, function(changes){
        changes.forEach(function(change) {
            if (typeof obj[change.name] == 'object') {
                observeNested(obj[change.name], callback);
            }
        });
        callback.apply(this, arguments);
    });
}

var model = {};

// Which we then observe
observeNested(model, function(changes){
    changes.forEach(function(change) {
        console.log(change.type, change.name, change.oldValue);
    });
});
setTimeout(function() {
    model.adding = 'a prop';
}, 200);
setTimeout(function() {
    model.nested = { obj: 'thing' };
}, 400);
setTimeout(function() {
    model.nested.obj = 'another thing';
}, 600);
{% endhighlight %}

Which shows:

{% highlight javascript %}
add adding undefined
add nested undefined
update obj thing 
{% endhighlight %}
