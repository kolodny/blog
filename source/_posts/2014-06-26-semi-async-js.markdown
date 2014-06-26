---
layout: post
title: "Semi-Sync JS"
date: 2014-06-26 11:32:44 -0400
comments: true
categories: js node promises
---

Writing async code is hard when not done properly. However there are ways to make it a breeze. One of them is to use promises and not have to worry about code being async or not.

I was looking around at different things and came across [koa](https://github.com/koajs/koa) and got inspired to try a similar idea.

The basic idea of Koa is that when you need some async functionality you yield an async thingy like a promise or something similar and then koa waits for it to resolve and then continues your program injecting the new value into the function.

Sounds complicated? Let's try an example (all examples will only run in FF unless Chrome decides to support generators)

Let's start with a simple generator.

<iframe width="100%" height="300" src="http://jsfiddle.net/2QsBH/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

As you can see the syntax is `function *() {}` and you need to instantiate a generator before you can use it.
For more info on the basics of generators see [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)

A cool thing about generators is that you can inject a value back into the generator function like so:

<iframe width="100%" height="300" src="http://jsfiddle.net/2QsBH/1/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

You actually can't inject something into the first `.next()` call, if you try you get this error: `TypeError: attempt to send "a" to newborn generator`

Using just these two concepts and the very basics about Promises, we can write something that:

1. Runs a generator until control is given back
2. If result if promise like, then resolve it and inject the resolved value back to the generator
3. Rinse and repeat.

Let's start to write just that. Here's what out program will look like:

{% highlight javascript %}
function *program() {
	var rand;
    rand = yield getRandAsync();
    console.log('first time around = ' + rand);
    rand = yield getRandAsync();
    console.log('second time around = ' + rand);
}

function getRandAsync() {
    return Promise.resolve(Math.random())
}

run(program());
{% endhighlight %}

Now all we need to do is write the `run` function:

{% highlight javascript %}
function run(gen) {

    step();

    function step(value) {
        var result = gen.next(value);
        if (result.value instanceof Promise) {
            result.value.then(step);
        }
    }

}
{% endhighlight %}

Putting that all together:

<iframe width="100%" height="300" src="http://jsfiddle.net/JHFd5/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Now a couple of nice thigs to have. Let's assume that sometime in the future `getRandAsync` is changed to just return a value and not a promise, we should handle that:

{% highlight javascript %}
function run(gen) {

    step();

    function step(value) {
        var result = gen.next(value);
        if (result.value instanceof Promise) {
            result.value.then(step);
        } else if (!result.done) {
        	step(result.value);
        }
    }

}
{% endhighlight %}

Easy enough. Now how would we handle concurrent async functions? Well the Promise object has a method `all` which handles it nicely, we just need to utilize it ([MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) ctrl+f `Promise.all`):

{% highlight javascript %}
function run(gen) {

    step();

    function step(value) {
        var result = gen.next(value);
        if (result.value instanceof Promise) {
            result.value.then(step);
        } else if (result.value instanceof Array) {
            Promise.all(result.value).then(step);
        } else if (!result.done) {
            step(result.value);
        }
    }

}
{% endhighlight %}

Now we can yield an array of promises to `run` and `Promise.all` will handle them. [The great thing about `Promise.all` is that it will handle an array of non promises just as well](http://kolodny.github.io/blog/blog/2014/04/23/future-proof-your-code-with-promises/)

putting that all together we get a very powerfull way to write javascript:

<iframe width="100%" height="300" src="http://jsfiddle.net/XKYVB/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

[**Gist**](https://gist.github.com/kolodny/6691380b57abd5b56251)