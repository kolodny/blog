---
layout: post
title: "Unoptimize DevTools For Easy Debugging"
date: 2014-05-19 11:42:09 -0400
comments: true
categories: javascript dev-tools
---

Every now and then I need to debug something in DevTools and I get this puzzling error:

[![ReferenceError](http://i.imgur.com/nRvfrMy.png)](http://i.imgur.com/nRvfrMy.png)

That's V8 optimizing the function and seeing that `thing` isn't used in the function and therefore not putting it into the closure scope. There are a couple of ways to break this optimization, one is to use `thing` somewhere in the debugging function, however the problem with this is that you must enumerate all the vars you want to inspect. An easier technique I've found is to make it impossible for V8 to know what vars will be used in the function. What that entails is to just have a blank `eval` in the function somewhere (I usually put in right before the debugger statement):

[![ReferenceError](http://i.imgur.com/wnFtgte.png)](http://i.imgur.com/wnFtgte.png)
