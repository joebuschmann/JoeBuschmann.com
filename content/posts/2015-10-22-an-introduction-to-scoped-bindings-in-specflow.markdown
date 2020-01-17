---
layout: post
title: An Introduction to Scoped Bindings in SpecFlow
date: '2015-10-22 12:28:04'
tags:
- specflow
- net
- c
---

One nice aspect of SpecFlow is the ability to scope bindings by feature title, scenario title, or tag. Normally bindings are global to the project, but a binding's scope can be restricted using the `Scope` attribute. I like to think of it as similar to the `private` and `public` class modifiers in C#.

Consider the Gherkin below.

<script src="https://gist.github.com/joebuschmann/26a8482d54d97c0761e9.js"></script>

It is a single feature with one scenario and two tags. One tag is at the feature level and the other at the scenario level.

The feature's corresponding steps can be scoped by:

**Scenario Title**

<script src="https://gist.github.com/joebuschmann/2df26540a2ae3a5fa369.js"></script>

**Scenario Tag**

<script src="https://gist.github.com/joebuschmann/359c5b34f2e6dd8cac67.js"></script>

**Feature Title**

<script src="https://gist.github.com/joebuschmann/08a29c27dd6204af4f5b.js"></script>

**Feature Tag**

<script src="https://gist.github.com/joebuschmann/0ae0cbd4163e053d1e40.js"></script>

In these snippets, the `Scope` attribute is applied at the class level, but you can also put them on individual methods. When the SpecFlow test runner executes the bindings, it will match the Gherkin to methods based on scoping rules. If there are multiple matches, the most restrictive match is used.

#### Same Gherkin, Different Bindings

To illustrate the usefulness of scoped bindings, consider the following feature that invokes a service with content serialized to XML, JSON, and Form-UrlEncoding. The text of the `When` step is the same in the Gherkin for all three scenarios, but each bound method is different. Without scoped bindings, this feature won't run because the SpecFlow runtime can't determine which method to execute.

<script src="https://gist.github.com/joebuschmann/72554b4c76974e70dedf.js"></script>

<script src="https://gist.github.com/joebuschmann/9db7552ff3a22209dfce.js"></script>

Scoped bindings can help. You could scope each method by scenario title.

<script src="https://gist.github.com/joebuschmann/ca4a46579de16c12768e.js"></script>

Or use tags.

<script src="https://gist.github.com/joebuschmann/9e9ae7dab81f27cfee61.js"></script>

<script src="https://gist.github.com/joebuschmann/fca3c06961a6dd0b7443.js"></script>

While this example works well for demonstrating the power of scoped bindings, I generally avoid coupling step definitions to Gherkin in all but the simplest features. In fact, this type of coupling has been identified as an [anti-pattern](https://github.com/cucumber/cucumber/wiki/Feature-Coupled-Step-Definitions-%28Antipattern%29).

So how would I fix this? That's the subject of [my next post](http://joebuschmann.com/specflow-tags-done-right/). In the meantime, you can read more about scoped bindings in the [SpecFlow documentation](http://www.specflow.org/documentation/Scoped-Bindings/).