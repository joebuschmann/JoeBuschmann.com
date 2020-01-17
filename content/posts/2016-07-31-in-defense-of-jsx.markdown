---
layout: post
title: In Defense of JSX
date: '2016-07-31 19:30:54'
tags:
- react
- jsx
- javascript
---

JSX is the much maligned JavaScript syntax extension that tells React how to build the UI. It gets translated into JavaScript which then builds out the virtual DOM. Later the virtual DOM is translated into the real DOM, and you've got a UI.

For some reason many people don't like JSX. Their criticism usually goes along the lines of: *JSX is ugly and you shouldn't mix the view and controller. You should have separation of concerns (aka MVC) for better reuse.* Well, after building two medium-sized React apps, I think mixing the view and controller is exactly what you should be doing. This approach leads to components which are highly reusable.

#### Seriously - Why MVC?

Before I continue my defense of JSX, a bit about MVC (or MVVM or MVP). The main goal of the MVC pattern is to separate the concerns of the UI into a model, view, and controller. The model is the data that drives the UI. The view is what gets rendered to the screen. The controller is the glue holding it all together.

Loose coupling is a purported benefit of MVC, but the reality is it leads to plenty of coupling between the model, view, and controller. That's been my experience anyway. Maybe I haven't worked on the right projects or with the right people. I've only seen views and controllers reused in the most trivial of cases. Just because MVC takes all the JavaScript out of the HTML doesn't mean the three parts can exist separately. They still need each other to produce an app.

**Don't get me wrong. I'm very much for separation of concerns. I just think MVC takes it too far.**

What about JSX? What are some of the critiques and why are they misguided?

#### JSX Couples the Model, View, and Controller

Dan Yoder over at Panda Strike wrote a little over a year ago about why [React is a terrible idea](https://www.pandastrike.com/posts/20150311-react-bad-idea). The post is well-written, and I recommend reading it. He makes makes good points especially about web components being the future. But I disagree with his criticism of JSX.

>**Sure, you can use JSX if you want, but that's the worst part of React anyway. JSX wants you to couple your view with the model and controller. It's a bad idea. Don't do that.** - Dan Yoder

Let's for a moment put aside the conventional wisdom of decoupling the model, view, and controller. React components, including JSX, can be great examples of separation of concerns. A textbox component is concerned only with rendering a textbox. An address component is concerned only with displaying and editing an address. They can be plugged into the app in any page. Each component defines a contract for receiving data. Breaking components down into MVC parts doesn't add much. In fact, it makes things worse by adding unnecessary complexity.

**I'll take reusable components over getting JavaScript out of my HTML.**

#### JSX Is Ugly

JSX is essentially XML with some extensions to embed code, otherwise it looks much like HTML. If JSX is ugly, then so is HTML. To me, JSX isn't any worse than HTML littered with *ng-** or *data-** attributes. Consider this example from the [AngularJS Developer Guide](https://docs.angularjs.org/guide/controller). Note `ng-controller`, `ng-click`, and the {% raw %}`{{ }}`{% endraw %} binding syntax.

```
<div ng-controller="SpicyController">
 <button ng-click="chiliSpicy()">Chili</button>
 <button ng-click="jalapenoSpicy()">Jalape├▒o</button>
 <p>The food is {{spice}} spicy!</p>
</div>
```

The equivalent JSX looks like this:

```
<div>
 <button onClick={chiliSpicy()}>Chili</button>
 <button onClick={jalapenoSpicy()}>Jalape├▒o</button>
 <p>The food is {this.state.spice} spicy!</p>
</div>
```

Visually, they're very similar. It's difficult to make the case that one is worse than the other.

#### className and htmlFor

Related to the previous point is the use of **className** and **htmlFor**. The JSX designers couldn't use the keywords **class** or **for** since they are also keywords in JavaScript. JSX introduces **className** and **htmlFor** to work around this. Sure, it feels a bit hacky but is it really that big of a deal?

#### Transpilation

JSX has a transpilation step, but isn't transpilation becoming more common anyway? If you're using Typescript or ES6, you'll need to transpile. Why not throw in JSX?

**UPDATE (8/29/2016)** - In my original post, I failed to emphasize a very important point. Unlike the equivalent HTML, JSX is checked for correctness by the transpiler. This eliminates many of the silent binding errors present in native HTML and frameworks like Angular.

#### MVC Is Needed for Testing

Below is an example of a unit test for a React button component that uses [Facebook's Jest, a unit testing framework](https://facebook.github.io/jest/). It supports rendering Font Awesome icons and a wait spinner onto a button. The test is as straightforward as they come. The component is rendered into a string of HTML using `ReactDOMServer.renderToStaticMarkup()` and then validated against the expected markup.

**How would MVC make this any easier?**

<script src="https://gist.github.com/joebuschmann/9e54dd93900df7be4e0110808eb2def3.js"></script>

#### Long Live JSX

Well, not really.

Eventually JSX critics will get their wish. React along with Angular, Knockout, and other UI libraries will be replaced with a better standards-based approach. [Web Components](https://www.google.com/search?q=web+component) is a set of W3C standards currently in working draft status. They include custom elements, HTML templates, HTML imports, and the shadow DOM. The specification is still in progress and [browser support is patchy](http://caniuse.com/#feat=custom-elements). In the meantime, React along with JSX feels like the best choice.