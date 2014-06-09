---
layout: post
title: "Require A Promise"
date: 2014-06-09 15:25:18 -0400
comments: true
categories: javascript node Promise
---

I was recently asked how to make a node module that needs to do a database lookup or something else before it can give you an API. The main problem is that `require` is synchronize and doesn't have a concept of callback or a mechanism to wait for a response The are a couple of ways to solve this, I'll first go through the standard solution and then how I would do it.

The standard way people solve this is by having the API take a callback:

{% highlight javascript %}
var gister = require('gister');

gister(function(err, create) {
	if (err) throw err;
	create(fileContents);
});
{% endhighlight %}

Now this is easy to read and makes a lot of sense, but you run into a problem of when you have more than one module like this:

{% highlight javascript %}
var gister = require('gister');
var snippeter = require('snippeter');
var apis = {};
var ticks = 2;

gister(function(err, create) {
	if (err) throw err;
	apis.gister = create;
	if (--ticks === 0) ready(apis);
});
snippeter(function(err, create) {
	if (err) throw err;
	apis.snippeter = create;
	if (--ticks === 0) ready(apis);
});
function ready(apis) {
	// apis.gister
	// apis.snippeter
}
{% endhighlight %}


This definitly doesn't scale, fortunately we can solve this using the magic of [**Promises**](http://kolodny.github.io/blog/blog/2014/04/23/future-proof-your-code-with-promises/)

File index.js:
{% highlight javascript %}
var Promise = require("bluebird");

var soon = require('./soon.js');

Promise.all([soon]).then(function(soon) {
	console.log(soon);
});
{% endhighlight %}

File soon.js
{% highlight javascript %}
var Promise = require("bluebird");

var ret = {
	obj: 'to return'
};

module.exports = new Promise(function(resolve, reject) {
	setTimeout(function() {
		if (Math.random() < .1) {
			console.log('about to reject');
			reject(new Error('Db error'));
			console.log('rejected');
		} else {
			console.log('about to resolve');
			resolve(ret);
			console.log('resolved');
		}
	}, 5000);
});

//module.exports = ret;
{% endhighlight %}


Now the cool thing about doing it this way is that we can switch back to a synchronize way of loading without having to change any code. We can simulate this by uncommenting the last line `//module.exports = ret;` and it will still run the same