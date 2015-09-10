---
layout: post
title: "Removing Css From Your Workflow"
date: 2015-08-26 12:00:38 -0400
comments: true
categories: javascript
---

I recently started a new project using webpack and babel and react. In the process I was able to eliminate css completely from my workflow. Here's how it used to look:

``` js
const MyComponent = React.createClass({
  render() {
    return (
      <ul className="my-list">
        {
          items.map(item => {
            return <li className="list-item">{item}</li>;
          })
        }
      </ul>
    );
  }
})
```

```
/* sass or less */
.my-list {
  font-size: 10px;
  position: relative;

  .list-item {
    color: red;
  }
}
```

or something like that. The problem with this approach is that all the css rules are global so you have to make sure not to step on anyone else's toes. Projects like [CSS Modules](http://glenmaddern.com/articles/css-modules) address this problem. Of course you can always namespace your modules with a root CSS class, but when you do have specific rules that are shared (invalid inputs for example), things start getting messy.

If you're already using babel and react then there's a similar solution you can use without installing anything else. The idea is to use [react inline style](https://facebook.github.io/react/tips/inline-styles.html). Using that we get rid of the everything being global and colliding with everything else problem So that's half the battle.

What about inheriting styles from other elements? Well instead of having a cascade happen, we can be explicit and declare exactly how we want to inherit other styles by using the [spread operator](https://github.com/sebmarkbage/ecmascript-rest-spread). Here's the final version of what the code would look like:

```
// style.js
export const ul {
  position: 'relative',
  fontSize: 10
};

export const li {
  ...ul,
  color: 'red'
};

// index.js
import {ul, li} from './style';
const MyComponent = React.createClass({
  render() {
    return (
      <ul style={ul}>
        {
          items.map(item => {
            return <li style={li}>{item}</li>;
          })
        }
      </ul>
    );
  }
})

```

A great thing about this is that looking at the class and the spreads on it, you can easily see which rules will override other rules based on the spread position, for example `o1:{a:1}, o2={b:2}, o3={a:3}, final={ ...o1, ...o2, ...o3 }` then final will equal `{a:3, b:2}`

This would make deep inheritance somewhat of a pain, but how often is deep inheritance actually needed? I suspect not that often