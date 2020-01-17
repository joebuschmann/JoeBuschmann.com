---
layout: post
title: Salesforce Lightning - Navigation Events in a Visualforce Page
date: '2017-01-16 10:00:00'
tags:
- salesforce
- lightning-component
- lightning
---

In my [previous post](http://joebuschmann.com/salesforce-lightning-hosting-a-component-in-visualforce), I covered how to use a Lightning component in a Visualforce (VF) page and mentioned an issue with navigation events. These events no longer work. The problem is VF pages are loaded into an iframe element in the Lightning Experience. Navigation events like *force:navigateToObjectHome* are handled by the one.app container in the parent frame. Events raised in the VF page stop at the iframe boundary and don't bubble up into the parent frame.

In this post, I discuss how to get navigation events working in a Lightning Out host like a VF page. My approach avoids the need to detect the host environment inside of a component. Instead, components continue to raise events as usual and are still decoupled from the environment. Events are handled explicitly in the Lightning Out host where the environment detection logic is isolated.

#### Raising Navigation Events

The first step is to get the navigation events firing properly. The Lightning code below will throw an error when run in a VF page.

<script src="https://gist.github.com/joebuschmann/e8e41873cbd9c24de8697a6758b501f9.js"></script>

The event variable is null because `$A.get()` can't find the event type. The issue is the *force.\** events are not recognized as a dependency by Lightning Out. Fortunately, the fix is straightforward. Add the force dependency to your Lightning dependency app.

<script src="https://gist.github.com/joebuschmann/1ff0f3302b9031d92a8e10a1abc3168e.js"></script>

With this change, the navigation events will fire properly in your components; however, there's still a problem. Nothing happens. This brings us to the second step which is to add event handlers in the VF host page.

#### Handling Navigation Events

At this point, events are firing, but nothing is listening. The app.one host comes with built-in listeners that properly handle all the force events, but a VF page is either isolated in its own iframe (Lightning Experience) or running outside of Lightning (Salesforce Classic). This means any VF page hosting a Lightning component will have to explicitly handle navigation events raised by the component and any dependencies. How it handles these events depends on whether the page is running in the Lightning Experience or Salesforce Classic.

The **Lightning Experience** (and Salesforce1) provides a JavaScript helper library called **sforce.one**. This library provides [methods for navigation](https://developer.salesforce.com/docs/atlas.en-us.salesforce1.meta/salesforce1/salesforce1_dev_jsapi_sforce_one.htm). Internally sforce.one uses [window.postMessage()](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) to send messages to the parent frame which in turn forwards the events to the app.one container.

**Salesforce Classic**, on the other hand, does not run app.one. Handlers have to detect this scenario and provide an alternate means to navigate. For example, `window.location` may be set with the proper URL corresponding to the type of event.

<script src="https://gist.github.com/joebuschmann/ed2d5569a18e85981b1668756093c88e.js"></script>

The code snippet above is from a VF page hosting a component named *customObjectEdit*. It adds a handler for *force:navigateToObjectHome* which detects the presence of the sforce.one object. If the object exists, `sforce.one.navigateToObjectHome` is invoked to go to the given sObject's home page. If it doesn't exist, the page is running in Classic mode, and `window.location` is used instead.

I like this approach because the Lightning component is decoupled from the host and communicates via events. The VF host page handles the events and reacts appropriately.

#### What About Other Events?

The sforce.one object is a nice wrapper library for navigation, but it doesn't include all the events available in the force namespace. For example, **force:showToast** has no corresponding method in sforce.one. You could provide a simple JavaScript alert, but an alert makes for a jarring user experience when a nice toast is expected.

There is a work-around for this that is not supported by Salesforce in any way. I discovered it while looking at the implementation of sforce.one. It uses an object **SfdcApp** which calls [window.postMessage()](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) to forward events to the app.one container. SfdcApp is not intended for public consumption, so Salesforce may change its implementation between releases.

#### Displaying a Toast Message

With this in mind, a new event handler can be added to the VF page to handle **force:showToast**.

<script src="https://gist.github.com/joebuschmann/7a2d15146aae92ee290aed4bb1fe3f52.js"></script>

Just add the snippet above to the `$Lightning.createComponent()` callback. The `postEventToOneApp` method will forward the event to the app.one container if the SfdcApp object exists (Lightning Experience) or invoke a fallback action when SfdcApp is not available (Salesforce Classic).

#### Wrapping Up

To sum things up, a VF page has to handle force events explicitly when hosting Lightning components. The sforce.one library makes this easier when running in the Lightning Experience or Salesforce1. In Classic mode, you'll need to write custom code for each event. Digging deeper, you can use the SfdcApp object to forward events to the one.app container, but this isn't supported by Salesforce.