---
layout: post
title: "Useful Snippets"
date: 2013-11-24 14:25:14 -0500
comments: true
categories: javascript dev-tools
---
I recently found [this useful page for devtools snippets](http://bgrins.github.io/devtools-snippets/)

I never really used snippets before (I do have a ton of bookmarklets though), and I wanted to
incorporate all the snippets on that page. There really isn't any built in way to do that, but since practically
everything in Chrome is exposed we can hack a way into it.

First off let's see what we have already. Open dev-tools and take a look at the snippets.

[![check for snippets](http://i.imgur.com/AI7ch4d.png)](http://i.imgur.com/AI7ch4d.png)

If it doesn't open in a new window then click the circled icon to open dev-tools in a new window.

Now you should have something that looks like this:

[![dev-tools in a new window](http://i.imgur.com/1xL13sh.png)](http://i.imgur.com/1xL13sh.png)

Now comes the cool part, we're going to open devtools on the devtools window. With the active window the
dev-tools window click `Ctrl + Shift + J` and you should see this:

[![inception dev-tools](http://i.imgur.com/dAO8Rh3.png)](http://i.imgur.com/dAO8Rh3.png)

Now go to the Resources tab and Local Storage, there may be more then one but it should be easy to figure
out which one you need, then go to the scriptSnippets object:

[![scriptSnippets](http://i.imgur.com/pcpPbsM.png)](http://i.imgur.com/pcpPbsM.png)

Hopefully you can see where this is going. Let's open up a dev-tools window on the second one by clicking
`Ctrl + Shift + J` on the second window, I resized a bit to make it more manageable:

[![](http://i.imgur.com/6vSHVH0.png)](http://i.imgur.com/6vSHVH0.png) 

Now let's head over to http://bgrins.github.io/devtools-snippets/ and scrape the snippets. Open up the
console on that page and paste:

{% highlight javascript %}
> var snippetsWrappers = document.getElementsByClassName('snippet-wrapper');
  var snippets = [];
  for (var i = 0; i < snippetsWrappers.length; i++) {
      snippets.push({
          name: snippetsWrappers[i].id,
          content: '//' + snippetsWrappers[i].childNodes[0].innerText.replace('\n', '\n//') +
              '\n' + snippetsWrappers[i].childNodes[1].innerText
      });
  }
  copy(JSON.stringify(snippets));
{% endhighlight %}

We now have a stringified version of the snippets in the system clipboard using the magic of
[copy on the console](../../../../2013/10/16/devtools-copy-to-clipboard/)

Now that we have that in the clipboard we can now close that window.  Now we need to merge in what we
have in the clipboard with the current snippets, one way to do that is with this (in the right hand
side window):

{% highlight javascript %}
> var snippetsToAdd = JSON.parse(prompt('Hit ctrl + v and enter'));
  var currentSnippets = JSON.parse(localStorage.scriptSnippets);
  var nextId = Math.max.apply(Math, currentSnippets.map(function(obj) { return obj.id })) + 1;
  for (var i = 0; i < snippetsToAdd.length; i++) {
      currentSnippets.push({
          name: snippetsToAdd[i].name,
          content: snippetsToAdd[i].content,
          id: nextId++
      });
  }
  localStorage.scriptSnippets = JSON.stringify(currentSnippets);
{% endhighlight %}

[![merging in snippets](http://i.imgur.com/ZXzD3Y3.png)](http://i.imgur.com/ZXzD3Y3.png)

You'll get a prompt and you need to paste in the contents of the clipboard. This was the easiest way I could
think of to share data between windows

If all went as planned then put focus on the bottom left window, hit F5 and you should now see:

[![Success!](http://i.imgur.com/PGoqrjB.png)](http://i.imgur.com/PGoqrjB.png)

You can see that the snippets are there in the top left window.

Play around while you're in there,  hopefully you'll learn about the internals of how dev-tools work
