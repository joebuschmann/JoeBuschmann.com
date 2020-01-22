---
layout: post
title: The Best Design Patterns
date: '2012-01-30 20:38:44'
tags:
- net
- c
- best-practices-2
- unit-testing
- patterns
---

The first time I read the Gang of Four Design Patterns book, I was impressed, no blown away, by the brilliance of what I was reading. The Visitor pattern utilizing double-dispatch. Brilliant. Chain of Responsibility. Why didn't I think of that? Adapter. Simple and powerful.

Of course I was itching to start implementing these patterns. In practice though they seemed to be applicable only in rare situations. Adapter, Builder, Factory, Strategy, were the most useful. The more exotic patterns rarely popped up, and when they did, they were misapplied or misused. For example, I've seen visitors used when double-dispatch wasn't necessary, and a simple iterator would have worked nicely.

Now that I have 10+ years of experience, I've come up with a list of 3 patterns (best practices really) that are crucial to any software project and trump any fancy pattern-based design.
<h3>Encapsulation</h3>
This is more of a best practice than a design pattern; however, I'm surprised at how many times data isn't properly protected. I prefer to initialize fields as read-only, if possible, just in case it is accidentally exposed. There are also subtle ways of exposing internal references. For example, in C# you could expose a private <em>List&lt;T&gt;</em> field using only a public getter, but the list can still be manipulated outside of the class. Instead, the list could be exposed as a read-only collection or as <em>IEnumerable&lt;T&gt;</em>.*
<h3>Request-Response Messages</h3>
I don't know of any formal name for this pattern, so I'm not sure what to call it. Essentially, it is designing classes whose methods take a single input or request object and return a single output or response object. There is no internal state at all. The data that each method needs is passed in with the request. This style is used frequently with service-based architectures like WCF; however, I like using it with any internal class. Without any side effects, the method is easy to understand and unit test.
<h3>Unit Tests!!</h3>
In my experience, most developers don't write unit tests with each code change or even a majority of code changes. I will often comment out lines of critical business logic at work and run the unit tests to determine if there is any coverage. Often they all pass. I view unit tests as documentation of how the system should behave. Proper coverage enables developers to refactor code with confidence; otherwise, they are more tentative, and the code base degrades over time.

Design patterns don't impress me like they used to, but I appreciate them. Still, I would take a system of static classes with static methods that is thoroughly unit tested over a system that is built with design patterns and lacks unit tests.

&nbsp;

&nbsp;

&nbsp;

\* Technically consumers of your class could cast the reference to a <em>List&lt;T&gt;</em> at which point they deserve what they get.