---
layout: post
title: Code This, Not That - SpecFlow Edition
date: '2014-12-29 20:06:20'
tags:
- c
- best-practices-2
- specflow
- code-this-not-that
---

In 2007 a different kind of diet book was published that took a concise approach to making the right food choices. Readers of the book were presented with good and bad selections at popular restaurants and told why the good options were better than the others. [Eat This, Not That!](http://www.amazon.com/This-Thousands-Simple-Swaps-Pounds/dp/B002M3SP6O) made changing your diet simple and realistic. No gimmicks like the Atkins diet or the current trend of cleansing. Just real options available in many restaurants.

I really like this approach to presenting good and bad choices. It conveys information in short and clear bites without long-winded prose to fill the pages. For my latest post, I decided to summarize all my past writing on SpecFlow into a guide written in the style of *Eat this, Not That!*

----

**WHEN WORKING WITH TABLES...**

<span style="color: #00FF00; font-weight: bold">Code this</span>

<script src="https://gist.github.com/joebuschmann/50e12313ef9023d16793.js"></script>

<span style="color: #FF0000; font-weight: bold">Not that</span>

<script src="https://gist.github.com/joebuschmann/b3ae7a70286fee811103.js"></script>

**Why?**

The `TechTalk.SpecFlow.Assist` namespace has a number of helper methods that make working with tables simple. You can create objects and lists with `CreateInstance<T>()` and `CreateSet<T>()` and perform comparisons with `CompareToInstance<T>` and `CompareToSet<T>`. These methods make explicitly accessing a table's rows and fields unnecessary in most cases. Your code will clean up nicely with all the boilerplate code gone.

----

**WHEN BUILDING COMPLEX TEST DATA...**

<span style="color: #00FF00; font-weight: bold">Code this</span>

<script src="https://gist.github.com/joebuschmann/258771b995d738ce270a.js"></script>

<script src="https://gist.github.com/joebuschmann/fcbe058b70b89d288292.js"></script>

<span style="color: #FF0000; font-weight: bold">Not that</span>

<script src="https://gist.github.com/joebuschmann/5f5eb9862b37d32f2a95.js"></script>

<script src="https://gist.github.com/joebuschmann/c1d100d0f14718a10fe4.js"></script>

**Why?**

If you find yourself building out complex test data using tables and stuffing more than just a simple value into a field, you may want to consider adding additional steps. In this example an entire address (lines 1 & 2, city, state and zip) is placed into a single table field with the parts delimited by a semi-colon. The corresponding step binding has to parse the text to build an `Address` object.

The binding becomes simpler when it is broken up into two steps: one for creating the customer and a second for adding the address. The new steps are easier for the stakeholders to read, and the error-prone parsing logic in the old step binding goes away.

----

**WHEN SHARING STATE BETWEEN BINDINGS...**

<span style="color: #00FF00; font-weight: bold">Code this</span>

<script src="https://gist.github.com/joebuschmann/a238cb7fcd234eb5432b.js"></script>

<span style="color: #FF0000; font-weight: bold">Not that</span>

<script src="https://gist.github.com/joebuschmann/a1c29ef9a418259dc1a0.js"></script>

**Why?**

The SpecFlow framework provides a way of saving state between binding invocations via a name/value collection in the `ScenarioContext.Current` singleton. Step bindings can add data to this collection for later steps to consume. While this approach is convenient, it can get messy for a large number of bindings. Just managing the keys can get out of hand.

A better approach is to use SpecFlow's built in [Dependency Injection](http://en.wikipedia.org/wiki/Dependency_injection) framework. SpecFlow's runtime engine will reuse the same binding instance throughout the lifetime of a scenario. You can take advantage of this by adding private member variables to share data between steps in the same class. For sharing data among different binding classes, you can define a [Data Transfer Object](http://en.wikipedia.org/wiki/Data_transfer_object) and inject it into each binding via the constructor. The runtime will know to include it and will reuse the same instance.

----

**WHEN PASSING BINDING PARAMETERS (PART 1)...**

<span style="color: #00FF00; font-weight: bold">Code this</span>

<script src="https://gist.github.com/joebuschmann/ade407b0ef96df81b7b7.js"></script>

<span style="color: #FF0000; font-weight: bold">Not that</span>

<script src="https://gist.github.com/joebuschmann/d20feb832cb48ea3a6f5.js"></script>

**Why?**

The SpecFlow engine treats a binding attribute's text as a regular expression. You can take full advantage of regex to transform the human-friendly Gherkin text into binding parameters. The Gherkin, `When I remove the product at index 5`, doesn't read well to a non-techie, but many developers will go with it because it's easy to extract the ordinal value of 5 into a parameter.

To fix this, you can rewrite the binding text into something more readable and still extract the numeric parameter. The regex literal `(\d+)(?:st|nd|rd|th)` converts *1st*, *2nd*, *3rd*, etc into its corresponding numeric value. The Gherkin can be rewritten to `When I remove the 5th product` which flows better.

----

**WHEN PASSING BINDING PARAMETERS (PART 2)...**

<span style="color: #00FF00; font-weight: bold">Code this</span>

<script src="https://gist.github.com/joebuschmann/c87ba3f0209567a0bf8a.js"></script>

<span style="color: #FF0000; font-weight: bold">Not that</span>

<script src="https://gist.github.com/joebuschmann/1485015753dfa9f0df60.js"></script>

**Why?**

Like the previous example, this binding can be rewritten with a different regular expression to make the Gherkin more readable and still extract a sensible method argument. You can take it one step further by leveraging step argument transformations to convert the text *from A-Z* into the proper enumeration value. The advantage of step argument transformations is they can be reused across bindings.

----

**WHEN USING NUMERIC IDENTIFIERS IN GHERKIN...**

<span style="color: #00FF00; font-weight: bold">Code this</span>

<script src="https://gist.github.com/joebuschmann/23072d552bf740e5d3d2.js"></script>

<script src="https://gist.github.com/joebuschmann/97314cf578cefff7ac01.js"></script>

<span style="color: #FF0000; font-weight: bold">Not that</span>

<script src="https://gist.github.com/joebuschmann/f91fc8d18469967084b6.js"></script>

<script src="https://gist.github.com/joebuschmann/0923456d6ccbd2c27edd.js"></script>

**Why?**

I limit the use of numeric identifiers in Gherkin steps as much as possible. They aren't descriptive and are difficult to update with a different data set.

A nice technique is to create a look-up table in the background section to map entity names to IDs. The subsequent steps can reference the entities by name, and in the step bindings, the names are used to look up the corresponding ID value. Later, if an ID value changes, only the look-up tables will need to be updated.

----

For more information on SpecFlow best practices, check out [my previous posts](http://joebuschmann.com/tag/specflow/) as well as the excellent [SpecFlow documentation](http://www.specflow.org/documentation/).