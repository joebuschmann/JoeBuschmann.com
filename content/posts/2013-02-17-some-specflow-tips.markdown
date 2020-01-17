---
layout: post
title: Some SpecFlow Tips
date: '2013-02-17 11:49:51'
tags:
- c
- specflow
---

UPDATE (11/18/2016):

>I've written a number of posts since this one was published that cover advanced SpecFlow topics like [composable steps](http://joebuschmann.com/refactoring-to-composable-specflow-steps/), [tags done right](http://joebuschmann.com/specflow-tags-done-right/), [managing state](http://joebuschmann.com/strategies-for-managing-state-in-specflow/), [useful regex](http://joebuschmann.com/useful-regex-for-specflow-bindings/), etc. For a list of all my SpecFlow writing, you can click on the SpecFlow tag or go here: http://joebuschmann.com/tag/specflow. SpecFlow is a fantastic tool that's changed the way I develop software, and I hope it does the same for you. Happy testing!

<hr />

Late last year a co-worker <a href="https://twitter.com/jaybrummels" target="_blank">Jay Brummels</a> introduced our development group to <a href="http://www.specflow.org/" target="_blank">SpecFlow</a> a .NET tool that enables behavior specifications from a domain expert or product owner to be bound to runnable test code. It has been a huge success, and we're quickly building up a suite of system tests and unit tests fronted by SpecFlow.

Although we're relatively new to SpecFlow, my group has identified some patterns or best practices to follow when building out tests.

<h3>Use Extension Methods to Manipulate ScenarioContext.Current</h3>

UPDATE (10/21/2017):

>Using ScenarioContext to store state is no longer recommended. Instead use dependency injection to share state between steps and bindings. For more information, see my post on [strategies for managing state](http://joebuschmann.com/strategies-for-managing-state-in-specflow/).

<em>ScenarioContext.Current</em> is a singleton that provides contextual information about the test and executing scenario block as well as a dictionary data structure for storing test data between scenario blocks. Below is an example of retrieving an object, manipulating it, and stashing it back in ScenarioContext dictionary.

<script src="https://gist.github.com/joebuschmann/5984a69af58ddd81bc429eaad558480f.js"></script>

Using the dictionary directly from step code has some disadvantages. First access to the data requires passing a string key which is not discoverable via Intellisense. Second creating an object may require multiple dictionary items to build it out. Having that logic in multiple places is prone to error.

To solve these issues, extension methods can be used to store and retrieve test data.

<script src="https://gist.github.com/joebuschmann/a5ef6563d44675d175982d6f10f58b88.js"></script>

<script src="https://gist.github.com/joebuschmann/9e37a43decb505ccad14a0bd3ee18df4.js"></script>

And to take things one step further, the creation of a complex object using multiple entries in the dictionary can be hidden behind an extension method.

<script src="https://gist.github.com/joebuschmann/917023f3f686c9dbd5fc270eb79bfa49.js"></script>

<h3>Bind Multiple Specification Attributes</h3>
After writing out a step specification, step definitions need to be generated and filled in with the implementation code. Whenever I notice the same step definition being used from different scenario blocks, I like to consolidate them into one method and add multiple step definition attributes. Also, I like to rename the method to something more concise. By default SpecFlow generates method names that match the step specification text, and they tend to be long and wordy.

<script src="https://gist.github.com/joebuschmann/d09cf0e7d01ccbe40b02548a3fb9f388.js"></script>

<script src="https://gist.github.com/joebuschmann/8399d007404790a8e61843217367fb85.js"></script>

<h3>Use Hooks to Execute Setup or Teardown Code</h3>
As with any other testing framework, SpecFlow provides a way of executing setup or teardown code for features, scenarios, steps, and test runs via <em>hook</em> classes. Hooks are associated to tests using tags which are keywords preceded by an <em>@</em> symbol.

At work my development group has created a series of system tests for an ordering system. Each test places an actual order in an integrated testing environment. After a test run, dozens of orders remain in the system taking up valuable space and resources. To solve this problem, a hook runs after each ordering scenario to cancel the order and remove it from the system.

<script src="https://gist.github.com/joebuschmann/9f3c4427c2bf7fe0a31dc7f3c0a32574.js"></script>

Tests scenarios that create orders can use the <em>@CancelOrder</em> tag to run the hook after the scenario completes.

<script src="https://gist.github.com/joebuschmann/a35da70cc254c885ba2aeed62bfd8a4d.js"></script>