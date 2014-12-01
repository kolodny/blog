---
layout: post
title: "Handling Optional Args In JS"
date: 2014-12-01 10:18:32 -0500
comments: true
categories: js
---

Often times I find myself writing functions which have optional args, while it's generally a better idea to pass an options object instead for mutliple arguments, three (maybe four) can still be considered an acceptable [arity](http://en.wikipedia.org/wiki/Arity).

Let's take a simple signature and implement some optional args.

```js
function require(name, deps, fn) {}
```

The way I started to deal with cases like this is as follows:

```js
function require(name, deps, fn) {
  var args = [].slice.call(argument);

  if (typeof args[0] !== 'string') {
    args.unshift(null);
  }

  if (!(args[1] instanceof Array)) {
    args.unshift([]);
  }

  if (typeof args[2] !== 'function') {
    args.unshift(function() {});
  }

  [name, deps, fn] = args; // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment

}
```

I find this the clearest way to deal with optional args and I find the logic very easy to follow.