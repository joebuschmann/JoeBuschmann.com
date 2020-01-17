---
layout: post
title: 'React: First Impressions'
date: '2016-05-16 13:30:13'
tags:
- react
- javascript
---

I'm a [React](https://facebook.github.io/react/) newbie who just completed my first React web app, a rewrite of an internal tool at work. Previously the UI was written in ASP.NET plus lots of JQuery on the client side. It worked well but was starting to show its age.

Actually this wasn't my first attempt at React. I started porting a web app written in [Knockout](http://knockoutjs.com/) to React last October. It was a side project and a good place to start. After a few days, I had to put it aside due to time constraints.

Fortunately, I decided to give it [five more minutes](https://signalvnoise.com/posts/3124-give-it-five-minutes), and this time I got it. Once I internalized the design philosophy behind React, using it became a pleasure.

#### React Components

When React proponents tout its benefits, they usually mention the efficiency of its virtual DOM or its a one-way reactive data flow. These features are nice, but for me, React's best aspect is how damn easy it makes building encapsulated components. Everything is a component from the topmost application down to simple widgets. The *right way* is the default rather than something to work toward.

React's approach is much easier to comprehend than say Angular's. For a newcomer Angular can be overwhelming. Do I build a Directive or a Component? According to the [doc](https://docs.angularjs.org/guide/component), an Angular Component is a "special kind of directive." Why have two? Why not have just Directives? Then there are [Controllers](https://docs.angularjs.org/guide/controller). And [Modules](https://docs.angularjs.org/guide/module).

React is dead simple. You have one way to organize your app: components, and that's it. Sure, there are stateful components and stateless functional components. But there's still only one concept to wrap your head around.

#### Components as Common JS Modules

React components lend themselves well to an organizational structure built around [Common JS](http://www.commonjs.org/). Components can be put into their own modules and bundled using [Browserify](http://browserify.org/) or [Webpack](https://webpack.github.io/). Modules are imported and exported with `require` and `module.exports`. The result is a neatly organized development experience.

Compare Common JS' management of dependencies to Angular's [dependency injection](https://docs.angularjs.org/guide/di). Since JavaScript isn't a strongly-typed language, Angular's DI layer uses function parameter names to resolve dependencies. That works well until you uglify your JavaScript, and then it completely breaks down. Angular implements a clunky work-around by allowing dependencies to be defined by an array of string values. These values need to be kept in sync with the function parameters. It works but doesn't feel natural. Common JS feels right.

#### MV-Whatever

Unlike almost every other UI library, React doesn't follow the MV* pattern. Instead it embeds the UI into each component via [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html). At first glance JSX seems strange, but in practice it helps make React components much more encapsulated and reusable. I really like it.

I started my career writing desktop applications in VB6. VB6 had ActiveX controls that you could compose into other ActiveX controls and drop them onto a form. They were highly reusable. Building UIs was easy and scaled well to a large team. React components work much the same way. They're easy to compose into more complex components.

One knock against ActiveX controls is they were difficult to test. There wasn't an easy way to mock a control's interaction with the graphics layer. This was one reason why MV* became so popular. By separating the view from the logic, you can provide a mock for the view and test the logic in the controller (or presenter or view model).

React is different from ActiveX controls because it doesn't interact directly with any native UI layer. Instead each component returns a lightweight representation of the UI called the [virtual DOM](https://facebook.github.io/react/docs/glossary.html). This virtual DOM is rendered as actual DOM elements with the [React-Dom library](https://www.npmjs.com/package/react-dom). Testing a React component is as easy as verifying that it produces the correct virtual DOM elements.

#### Learning Curve

I've had little web development experience considering my 16 year career. As I mentioned earlier, my first React app was a complete rewrite of an internal work tool. I started it on January 4. The date sticks with me because React and its tools were new territory. In four months I was able to learn React, navigate the Node/NPM/JS tooling landscape, write an app, and along the way build out a library of reusable components. React enabled me to do that. I doubt I could have worked so quickly in Angular, Ember, or Knockout.

#### What Didn't Go Well

One thing I struggled with was how to manage state. React offers no guidance on state management. It's just a view library. Terms like Flux and Redux kept popping up during my research, but I ignored them so I wouldn't get overwhelmed.

State management is one of those problems that snuck up on me. In the early stages, the app was small, and state was easy to manage. As the app grew, I encapsulated smaller pieces of data into a few objects. Actions on the objects became methods. I was pleased with the proper OO design...until these objects had to be passed through layers of components to get to where they were needed. To solve this, I introduced a service bus using Node's EventEmitter. The end result was state encapsulated in several chunky JavaScript objects with some actions defined as methods and others as events.

This state mishmash works for now but won't scale. A better approach seems to be [Redux](http://redux.js.org). It provides a consistent predictable way to manage application state.

#### The Right Way

Today I'm all in with React. I recently began my second app, and I look forward to refining my experience. I love the simplicity and reusability of React components. They can be used to build more complex components which in turn can be composed into ever more complex components. Eventually state is thrown into the mix and you have a React application. That's the beauty of React. The *right way* is the default.