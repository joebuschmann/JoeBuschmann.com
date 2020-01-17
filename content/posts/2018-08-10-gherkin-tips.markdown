---
layout: post
title: Gherkin Tips
date: '2018-08-10 13:22:54'
tags:
- specflow
- gherkin
---

> \[Gherkin] is a Business Readable, Domain Specific Language that lets you describe software's behaviour without detailing how that behaviour is implemented.
> \- Gherkin Wiki

These Gherkin best practices were originally included in an early draft of a [talk I gave on SpecFlow](https://joebuschmann.github.io/scaling-specflow/). Ultimately, I took them out because they didn't fit well with the topic, so I put them into a blog post.

### Table values should be atomic

Bindings that take a table argument will almost always convert the table to a C# object for easy manipulation. With this in mind, Gherkin steps should make using tables in code straightforward, meaning values should be as atomic as possible. The sample below uses a single table to create a customer and an address. Each table field, Name and Address, crams in complex data that the binding has to parse. Parsing table values is an indication they should be broken up into smaller pieces.

<script src="https://gist.github.com/joebuschmann/670e31296271edb636345b33d2446c06.js"></script>

A better approach is to create two steps: one for the customer and one for the address. Each discrete piece of data should have its own column in the table. The name is broken up into Salutation, First Name, and Last Name columns. The address step has columns for Line 1, City, State, and Zipcode.

<script src="https://gist.github.com/joebuschmann/30bd0d5c3f6fd8eb74f3b4b02a2d2c5b.js"></script>

### Use natural language instead of tech speak

By design, Gherkin should use natural language accessible to business owners. Tech speak has a tendency to sneak in over time. This example contains text referring to an index of a list or array. It makes sense to engineers but may be confusing to the business.

<script src="https://gist.github.com/joebuschmann/c443c2ca10762dfa380487258294b7bf.js"></script>

You can take advantage of regular expressions to refactor this step to use the more natural language, "the 5th product", and still extract the index value in the binding.

<script src="https://gist.github.com/joebuschmann/d9c473b92a55df15520834edd42a5f19.js"></script>

### Avoid numeric IDs

Many integration test suites will use a well known data set to drive the tests. These data may be in a file, database, or available via a web service. Over time test engineers will memorize the numeric identifiers for entities used in common scenarios and start to refer to the entity IDs rather than a descriptive name. Try to avoid using these IDs in Gherkin.

<script src="https://gist.github.com/joebuschmann/0be725e59550c3f46327f9e8539112a2.js"></script>

Outside of engineering, no one knows what product 46 is. You can make this better by defining a map of descriptive names to IDs in a feature's background steps. The Gherkin uses the name while the steps map the incoming name to its ID. In this case, product 46 maps to the more descriptive identifier "iPhone 6".

<script src="https://gist.github.com/joebuschmann/6f325376516441241b5d0f92bfd92e58.js"></script>

### Put rules and equations in the feature file description

The feature header and description is a good place to define the rules and equations under test. If you're validating an insurance quoting engine, you can describe its rules at the top and avoid cluttering the scenarios with details of the algorithm.

<script src="https://gist.github.com/joebuschmann/c547369d48e03edabc85d219aa1e42e2.js"></script>

An obvious and lazy description helps no one. You might as well leave it out. A good description describes the algorithm you're testing. In this case, the insurance quote engine starts with a baseline price and adjusts it up or down based on the insured's individual risk and their neighbor's risk.

<script src="https://gist.github.com/joebuschmann/ed7a99fc92647b67a976f418b8a5a016.js"></script>

The following scenarios simply plug in the numbers and validate the final quote.

<script src="https://gist.github.com/joebuschmann/8e41d7da062d678dcf594337bbe0e222.js"></script>

<hr />

There are many more tips for writing effective Gherkin, but I've found these to be the most unique and useful. For more tips, check out the links below.

* [15 Expert Tips for Using Cucumber](https://www.engineyard.com/blog/15-expert-tips-for-using-cucumber)
* [9 tips for improving Cucumber test readability](https://www.foreach.be/blog/9-tips-improving-cucumber-test-readability)
* [BDD 101: Writing Good Gherkin](https://automationpanda.com/2017/01/30/bdd-101-writing-good-gherkin/)