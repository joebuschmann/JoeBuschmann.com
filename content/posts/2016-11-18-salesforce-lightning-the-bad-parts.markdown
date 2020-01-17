---
layout: post
title: Salesforce Lightning - The Bad Parts
date: '2016-11-18 14:25:00'
tags:
- salesforce
- lightning-component
- lightning
---

I try to avoid writing rants, but after a rough day, I needed to get this out. For the last two months, I've been working with the Salesforce Lightning Framework. It is the most frustrating JavaScript framework I've ever used. Unlike my experiences with Angular, React, and Knockout, I feel like Lightning is constantly fighting me. Technically it is built on the open source Aura framework, so this is an Aura critique as well. In my mind though the two are one and the same.

So here are my gripes with Lightning in no particular order.

#### Local Variables as Attributes

Lightning requires you to create state variables for a component in a declarative manner. The **&lt;aura:attribute /&gt;** markup declares which types of data can be passed into a component. With an access modifier value of "private", an attribute essentially becomes a local variable whose value is only visible to the component. Manupulating an attribute's value is restricted to the `component.get()` and `component.set()` methods. The result is a very awkward way of manipulating data.

I tried to avoid attributes by storing data in the helper object until I realized a single helper instance is shared across all instances of a component. This can be useful in scenarios that call for static data, but if you need a private variable, you're stuck with attributes.

```
	<!-- State is declared as Aura attributes. -->
	<!-- Attributes passed into the component via markup. -->
	<!-- They are accessed in code using component.get() and component.set(). -->
	<aura:attribute name="suggestions" type="Object[]" default="[]" />
	<aura:attribute name="value" type="Object" />
	<aura:attribute name="text" type="String" />
	
	<!-- This is a private to the component. -->
	<!-- Even if you're not using it in an expression within markup, you still need to do this. -->
	<aura:attribute name="cache" type="Object" access="private" />
```

#### Too Much Declarative Coding

In addition to declaring attributes, Lightning markup also comes with an [expression language](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/expr_overview.htm) (that is not JavaScript) and constructs like `<aura:if />`, `<aura:iteration />`, `<aura:handler />`, etc. You have to learn this expression language and the markup in order to do things that are trivial in JavaScript.

All this declarative coding gets messy fast, and it doesn't build on what you already know. I much prefer React's approach which is to incorporate JavaScript into the framework as much as possible. Sure you have JSX, but the expression language in JSX *is* JavaScript. React builds on the language and constructs you know rather than inventing new ones.

#### Renderer Methods

To make things more confusing, overriding renderer methods involves no markup at all. Salesforce decided to do away with the declarative style and go with JavaScript for rendering. Renderer overrides are simply JavaScript methods in the component's renderer.js file. I like this approach but would have preferred consistency. Why not this: `<aura:handler name="render" value="{!this}" action="{!c.doRender}"/>`? Ugly but consistent.

#### Explicit Typing

Lightning requires explicit types for attribute declarations which is odd considering it runs in a non-explicit dynamic environment. I mentioned earlier that the React UI library builds on JavaScript. React embraces its runtime environment rather than trying to fix it. With Lightning, I get the opposite feeling. Its approach is to try to fix JavaScript.

#### Superficial Separation into a Controller and Helper

I don't know what the point of the helper file is. The Lightning documentation encourages developers to keep the controller lean and put the bulk of a component's logic into the helper file. In practice the controller becomes a useless middle-man that just passes stuff to the helper. And you know what they say about cutting out the middle-man. Why not just have one file?

#### No Common JS Modules

Salesforce decided to ignore Common JS and just about every other modern JavaScript construct. Lightning components are wrapped in IIFEs at runtime, but it would have been nice if Saleforce embraced Common JS as a way of encapsulating component code.

#### No Callbacks as Attributes

Higher order functions are common in JavaScript, and as a result, callbacks are fundamental to pretty much every library I've used. Lightning replaces the simple callback with the more complex component event idea. Events certainly have their place in pub-sub scenarios, but sometimes a callback is more appropriate. It would be nice if attributes had an additional type of "callback" or "function". Unfortunately, this doesn't exist. You have to declare an event type and register an event of this type before you can pass a callback as an attribute. This is too much code for what is a simple concept in other frameworks.

#### Embracing the Lightning Way

While I don't enjoy Lightning, I will be using it for the foreseeable future. I'm a big believer in providing value. Salesforce is a significant investment for my company, and that's where I can best provide value. So I'll continue plugging away and embrace the Lightning way.