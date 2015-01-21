---
layout: post
title: "Solving The \"Where Did This Tab Come From?\" Question"
date: 2015-01-20 20:07:03 -0500
comments: true
categories: dev-tools
---


Sometimes I find myself with long lived tabs and I don't always remember how I wound up on some of those pages.

Here's a neat little trick I discovered today while trying to find out if I should share an article with someone or if they already shared it with me.

First things first, [open dev-tools](https://developer.chrome.com/devtools#access)

Then go to the network panel ![network panel](http://i.imgur.com/PFYhqel.png)

Now refresh the page and scroll to the top of the list of resources and click the top one ![referer](http://i.imgur.com/NRqEsZd.png)

Under the referer request header you should see how you wound up on that page.
