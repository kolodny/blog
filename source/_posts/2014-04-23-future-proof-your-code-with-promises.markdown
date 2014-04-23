---
layout: post
title: "Future Proof Your Code With Promises and Promise.all"
date: 2014-04-23 11:50:39 -0400
comments: true
categories: javascript promises
---

When starting out with async code there are a few option to manage the callback hell. The one gaining the most traction is promises.

<small>I starting writing a promise-like library when I first heard of promises: [WTTT (When This Then That)](https://github.com/kolodny/wttt)</small>

I was playing around with socket.io and had to do some async code as follows:

{% highlight javascript %}
socket.on('join', function(id, name, callback) {
  socket.join(id);
  socket.name = name;
  var people = _.map(io.sockets.clients(room), function(socket) {
    return {
      id: socket.id,
      name: socket.name,
    };
  });
  callback(people);
});
{% endhighlight %}

Looking at the [docs](http://socket.io/#how-to-use) show that I should be using an async version as follows:

{% highlight javascript %}
socket.on('join', function(id, name, callback) {
  socket.join(id) // this is still a sync function
  socket.set('name', name, function() {
    var people = ???;
    callback(people);
  });
});
{% endhighlight %}

As you can see there's some magic that we need to do. Enter promises.

The basic usage is as follows

{% highlight javascript %}
var promise = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve("It's now two seconds later");
  }, 2000);
});
promise.then(function(value) {
  // value === "It's now two seconds later";
});
{% endhighlight %}

The library that I'm using is [bluebird](https://github.com/petkaantonov/bluebird), they provide a `Promise.promisifyAll` method which takes an object
and converts all it's methods that have node style callbacks (`function callback(err, result) {...}`) as a last argument to promises

Promises by itself is not really that usefull, but the real power comes when you need to do many things at once, that's what Promise.all is for

{% highlight javascript %}
var resolveTo = function(thing) {
  return new Promise(function(resolve) {
    resolve(thing);
  });
};
var promises = [resolveTo('Apple'), resolveTo('Orange')]
Promise.all(promises).then(function(results) {
  // results === ['Apple', 'Orange'];
});
{% endhighlight %}

`Promise.all` also can take a non-thenable value and use it as is for example
{% highlight javascript %}
var promises = [resolveTo('Apple'), resolveTo('Orange'), 42]
Promise.all(promises).then(function(results) {
  // results === ['Apple', 'Orange', 42];
});
{% endhighlight %}

Armed with this we can now write the above socket code as follows
{% highlight javascript %}
socket.on('join', function(id, name, callback) {
  socket.join(id) // this is still a sync function
  socket.setAsync('name').then(function() {
    getPeopleAsync(id).then(function(people) {
      callback(people);
    });
  });
});
function getPeopleAsync(roomId) {
  return new Promise(function(resolve, reject) {
    var people = io.sockets.clients(room);
    var promises = [];
    people.forEach(function(socket) {
      promises.push(
        new Promise(function(resolveInner) {
          Promise.all([socket.id, socket.getAsync('name')]).then(function(results) {
            resolveInner({id: results[0], name: results[1]});
          });
        });
      );
    });
    Promise.all(promises).then(function(people) {
      resolve(people);
    });
  });
}
{% endhighlight %}

Which `getPeopleAsync` can be refactored to:
{% highlight javascript %}
function getPeopleAsync(roomId) {
  return new Promise(function(resolve, reject) {
    var people = io.sockets.clients(room);
    var promises = [];
    people.forEach(function(socket) {
      promises.push(
        new Promise(function(resolveInner) {
          Promise.all([socket.id, socket.getAsync('name')]).then(function(results) {
            resolveInner({id: results[0], name: results[1]});
          })
        })
      );
    });
    Promise.all(promises).then(resolve);
  });
}
{% endhighlight %}
Which `getPeopleAsync` can be refactored to:
{% highlight javascript %}
function getPeopleAsync(roomId) {
  var people = io.sockets.clients(room);
  var promises = [];
  people.forEach(function(socket) {
    promises.push(
      new Promise(function(resolve) {
        Promise.all([socket.id, socket.getAsync('name')]).then(function(results) {
          resolve({id: results[0], name: results[1]});
        })
      })
    );
  });
  return Promise.all(promises)
}
{% endhighlight %}

Now this lets us use Promise, notably Promise.all without having to worry about code ever changing from sync to async, consider:
{% highlight javascript %}
Promise.all(socket.getAsync('name'), socket.getAsync('joined'), serverId, personRecord).then(function(results) {
  // do things with results
});
{% endhighlight %}
and
{% highlight javascript %}
Promise.all(socket.getAsync('name'), socket.getAsync('joined'), serverId, db.getPersonAsync(personId)).then(function(results) {
  // do things with results
  // absolutely no changes needed here
});
{% endhighlight %}

Happy coding
