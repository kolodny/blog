---
layout: post
title: "Evaling With Style"
date: 2014-10-30 08:19:10 -0400
comments: true
categories: js
---

A couple of projects I've been working on, required me to turn strings into functions (templating, custom require, [mocha clone](https://github.com/kolodny/m2)) and it didn't seem like there was a good way to do it with things I needed in scope.

Let's take a look at a couple of ways to do it.

There's [John Resig's microtemplating way](http://ejohn.org/blog/javascript-micro-templating/) which is something like this:

```js
var str = 'hello <%= obj.name %>, upperized: <%= obj.name.toUpperCase() %>';
var ctx = { name: 'Moshe' };
var go = function(str) {
  return new Function('obj',
    [
      'var p = [];',
      'p.push(',
        '"' + str.replace(/<%= (.*?) %>/g, '", $1, "') + '"',
      ');',
      'return p.join("")',
    ].join('')
  )
}
console.log(go(str)(ctx))
```

The issue with this way is that this doesn't scale, since you have to keep adding more params of things you need scope for. For example if we wanted to add helper functions:


```js
var str = 'hello <%= obj.name %>, upperized: <%= helpers.up(obj.name) %>';
var helpers = {up: function(str) { return str.toUpperCase(); }};
var ctx = { name: 'Moshe' };
var go = function(str) {
  return new Function('obj', 'helpers',
    [
      'var p = [];',
      'p.push(',
        '"' + str.replace(/<%= (.*?) %>/g, '", $1, "') + '"',
      ');',
      'return p.join("")',
    ].join('')
  )
}
go(str)(ctx, helpers)
```

You can see that this doesn't really work as more things are needed.

Another way is to use `with` to put things in scope:

```js
var str = 'hello <%= obj.name %>, upperized: <%= h.up(obj.name) %>';
var helpers = {up: function(str) { return str.toUpperCase(); }};
var ctx = { name: 'Moshe' };
var go = function(str) {
  return new Function('obj', 'expose',
    [
      'with (expose) {',
        'var p = [];',
        'p.push(',
          '"' + str.replace(/<%= (.*?) %>/g, '", $1, "') + '"',
        ');',
        'return p.join("")',
      '}',
    ].join('')
  )
}
go(str)(ctx, {h:helpers})
```

This works really well but there's a [huge performance hit from using `with`](http://jsperf.com/with-statement/4)

If you're using node there's a [vm module](http://nodejs.org/api/vm.html) which is really what we're looking for, but I wanted something I could also use in the browser.

After playing around with this problem, I came up with the following solution:

```js
function eval2(str, context, expose) {
  var exposeKeys = [];
  var exposeValues = [];
  for (var i in expose) {
    if (Object.hasOwnProperty.call(expose, i)) {
      exposeKeys.push(i);
      exposeValues.push(expose[i]);
    }
  }
  return (new Function(
    'return function(' + exposeKeys + '){return function(){' + str + '}.bind(this)}'
  ))().apply(context, exposeValues);
}
```

You would use something like that as follows:

```js
var ctx = {};
var expose = {up: function(str) { return str.toUpperCase() }};
eval2('console.log(arguments)', ctx, expose)('Look! Even has args!')
```

Let's go through how this works.

Consider this:

```js
var r = new Function('return function inner() {}');
console.log(r.toString()); // 'function anonymous() {\nreturn function inner() {}\n}'
console.log(r().toString()) // 'function inner() {}'

var s = new Function('return function(a,b,c){console.log(a,b,c)}');
console.log(s.toString()) // 'function anonymous() {\nreturn function(a,b,c){console.log(a,b,c)}\n} '
console.log(s().toString()) // 'function (a,b,c){console.log(a,b,c)}'
s()('aa', 'bb', 'cc') // logs 'aa', 'bb', 'cc'
```

Now going to back to the way we use it:

```js
var code = 'console.log(this)';
var ctx = {the: 'context'};
var fn = (new Function('return function() {' + code + '}'))().call(ctx);
```

If you wanted to have some stuff in scope:

```js
var code = 'log(this); hey(this.the)';
var ctx = {the: 'context'};
var params = ['log', 'hey']; // [].toString joins on comma by default so it just works
var applies = [function(str) { console.log(str) }, alert];
var fn = (new Function('return function(' + params + ') {' + code + '}'))().apply(ctx, applies);
```

And if you wanted to curry it just wrap it in another function:

```js
var code = 'log(this); hey(this.the)';
var ctx = {the: 'context'};
var params = ['log', 'hey']; // [].toString joins on comma by default so it just works
var applies = [function(str) { console.log(str) }, alert];
var fn = (new Function('return function(' + params + ') {return function(){' + code + '}.bind(this)}'))().apply(ctx, applies);
```

I didn't really run any benchmarks but I suspect that this is pretty performant and once you wrap your head around it, is actually pretty simple.

Happy Coding!