---
layout: post
title: 'React By Example: Mixins'
date: '2016-06-16 02:44:00'
tags:
- javascript
- react
---

Components are React's preferred reuse mechanism, but it's not the only one. Sometimes different components share the same functions. It may be awkward to wrap these cross-cutting concerns in a higher order component, or the common code may need access to a component's state. In these scenarios, React **mixins** are useful.

Before I continue, I should note mixins seem to be [on the way out](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750#.3umfys9zz). The React team is focused on making components more powerful versus continuing to build on the mixin concept. In fact, React's ES6 syntax doesn't support them at all.

That's too bad because I like [mixins](https://facebook.github.io/react/docs/reusable-components.html#mixins). I feel they're easier to use than higher order components. So until the React team rips them out of the codebase, [damn the torpedos, full speed ahead](https://en.wikipedia.org/wiki/Battle_of_Mobile_Bay#Damn_the_torpedoes)!

#### Detecting Clicks Outside of a Component

One asset that came out of a recent React app is a mixin to detect clicks outside of a component. It works by invoking a callback whenever a click event fires outside of the target component or one of its child elements. I use it to close popup panels like the options list of a custom dropdown or a modal dialog.

<script src="https://gist.github.com/joebuschmann/9a1944ea8c8f36557234.js"></script>

#### How It Works

The ClickAway mixin works by taking advantage of a couple of React's [component lifecycle methods](https://facebook.github.io/react/docs/component-specs.html#lifecycle-methods). It implements `componentDidMount` to attach a unique ID to the component's top level DOM node. The ID is used to identify the component when tracking click events.

```
var elementId = id++;
var element = ReactDOM.findDOMNode(self);
element.setAttribute(attributeName, elementId);
```

Next, an event listener is added to the document node to monitor all mouse clicks (or taps for mobile) on the page. When a click event occurs, the handler locates the DOM node where the event originated and searches it and all its parent nodes for the ID added by the mixin. If the ID is found, the click event is ignored. If not, the onClickAway callback is invoked indicating the event was triggered outside of the component. For something like a popup menu, the callback would close the menu.

```
self.clickAway = function (event) {
    if (!elementWithIdExists(event.target, elementId) && self.onClickAway) {
        self.onClickAway(event);
    }
};

window.document.addEventListener('click', self.clickAway);
```

Finally, the click event listener is removed in the `componentWillUnmount` lifecycle method.

```
componentWillUnmount: function () {
    window.document.removeEventListener('click', this.clickAway);
}
```

#### How to Use It

A quick note before I continue. I use a module bundler like [Browserify](http://browserify.org/) or [Webpack](https://webpack.github.io/) to manage React apps. Module bundlers enable dependency management via `require` and `module.exports` a&#768; la Node.js but on the client. If you've never used one, I recommend it. I went with Browserify because it's simpler to set up. ClickAway.js is organized as a module, and you'll need a bundler to consume it.

ClickAway is useful for dropdowns and modal panels which need to close when the user clicks elsewhere on the page.

To use it, first import it via `require`.

```
var ClickAway = require('../mixin/clickAway');
```

Mixins are included in a component by adding a mixins property to the component's class specification. Its value is set to an array of mixin components.

```
// Mixin to detect clicks outside of this component.
mixins: [ClickAway],
```

Finally, provide an implementation for the `onClickAway` callback. In the case of the popup panel, it invokes an `onClose` callback to notify the parent component it should be closed.

```
// Close the panel when the user clicks outside of the component.
onClickAway: function () {
    if (this.props.onClose) {
        this.props.onClose();
    }
},
```

The full source is below.

<script src="https://gist.github.com/joebuschmann/496739e921fa4fc6d0c4d76aa8c37a70.js"></script>

#### A Better Way?

I'm sure there's a better way to detect outside clicks. Perhaps with some kind of hit test? If you have any suggestions, please let me know. In the meantime, this mixin works on Firefox, Chrome, and IE 9+.