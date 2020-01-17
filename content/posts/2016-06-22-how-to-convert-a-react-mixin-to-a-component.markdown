---
layout: post
title: How to Convert a React Mixin to a Component
date: '2016-06-22 13:39:42'
tags:
- react
- javascript
---

In a [previous post](http://joebuschmann.com/react-by-example-mixins/) I covered an example of a React **mixin** which I called ClickAway that detected clicks anywhere outside of a component. I also mentioned mixins are not React's preferred method of reuse. Composition with components is the way to go, and [mixins are on the way out](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750#.rdurthwrj).

With this in mind, I decided to convert the ClickAway mixin into a component. I was skeptical about how clean the new implementation would be, but it turned out well. As you'll see, the equivalent component version better enables composition and is more explicit.

#### ClickAway as a Mixin

First, here's the mixin code from my previous post. It exports an object with two React lifecycle methods: componentDidMount and componentWillUnmount. These methods set up and tear down the click detection code.

<script src="https://gist.github.com/joebuschmann/9a1944ea8c8f36557234.js"></script>

And here's how you would use it. Notice the `onClickAway` method in the popup component is required by the mixin, but there's nothing explicitly defining this requirement. This is one of the weaknesses of a mixin. There is no explicit contract between it and the target component.

<script src="https://gist.github.com/joebuschmann/496739e921fa4fc6d0c4d76aa8c37a70.js"></script>

#### ClickAway as a Component

Migrating ClickAway to a component solves two issues. One, it enables composition, and two, it uses props and PropTypes to define an explicit contract.

Most of the code doesn't change much. The helper functions `hasId` and `elementWithIdExists` port over as is. I used `React.createClass` with the same implementations for the `componentDidMount` and `componentWillUnmount` lifecycle methods.

There are two differences. First, as a component, it needs to have a `render` method. In this case it's simple. ClickAway doesn't alter the look of the UI so it simply returns its child elements.

```
    render: function () {
        return this.props.children;
    }
```

Second, the `onClickAway` callback is passed in as a prop value. This is more explicit than the mixin implementation where it is a method of the component. PropTypes further define it as a required function.

```
    propTypes: {
        onClickAway: React.PropTypes.func.isRequired
    }
```

```
    self.clickAway = function (event) {
        if (!elementWithIdExists(event.target, elementId) &&
            self.props.onClickAway) {
            self.props.onClickAway(event);
        }
    };
```

Here's the full implementation.

<script src="https://gist.github.com/joebuschmann/ec30f22d9bb5a83db6c73691374417db.js"></script>

#### Using the Component

Now that it's a component, ClickAway needs to be composed with other components. In the case of the popup, it wraps the return value of `render()` with a `<ClickAway>` JSX element. It also provides the `onClickAway` callback as a prop value.

```
<ClickAway onClickAway={this.props.onClose}>
    <div {...elementProps}>
        {this.props.children}
    </div>
</ClickAway>
```
Below is the new implementation of the popup component.

<script src="https://gist.github.com/joebuschmann/1c75521aa233fd41fed881eaa8aa69f3.js"></script>

#### Components > Mixins

This exercise convinced me that components are indeed better than mixins. They enable composition and have an explicit contract. My next step is to convert the mixins in my current projects.


