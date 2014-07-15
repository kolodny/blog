---
layout: post
title: "Writing Async Javascript Code in Sync Format - Part 2"
date: 2014-07-15 12:33:23 -0400
comments: true
categories: 
---

[This is a continuation of Writing Async Javascript Code in Sync Format - Part 1](/blog/2014/06/26/semi-async-js/)

There's a new es6 feature called [Destructuring Assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) that looks like this:

```
var [a, b] = ['a', 'b'];
a === 'a';
b === 'b';
var {q:q, r:r} = {q: 'q', r: 'r'};
q === 'q';
r === 'r';
```

Keep in mind that objects can mimic arrays so this works:

```
var [x,y] = {0: 'x', 1: 'y'};
x === 'x';
y === 'y';
```

There's also a shortcut available for assigning to objects in es6 where `var {x:x, y:y}` can be written as `var {x, y}` This means we could have written 

```
var {q, r} = {q: 'q', r: 'r'};
q === 'q';
r === 'r';
```

Using this knowledge and what we did in the first part we can write really cool concurrent code:

<iframe width="100%" height="300" src="http://jsfiddle.net/SLTx9/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>


`Promise.props` is a bluebird specific function that does what `Promise.all` except on objects. [If you're using native Promises it doesn't take much to polyfill it](https://github.com/kolodny/promise-utils/blob/master/promise-props.js)