---
layout: post
title: Overusing Generics in .NET
date: '2014-10-10 12:31:09'
tags:
- c
- best-practices-2
---

Generic types were a great addition to C# 2.0, but they are occasionally overused. There are times where calling object.GetType() or passing the type as an argument are sufficient.

A good example can be found in the Specflow source code. The `TechTalk.SpecFlow.Assist.InstanceComparisonExtensionMethods` class contains a useful extension method `CompareToInstance<T>()` which takes a table of expected property values and compares them against an object.

<script src="https://gist.github.com/joebuschmann/0efa93da64b19e6c2576.js"></script>

The generic type is completely unnecessary in this case. `T` is used as the type of the instance argument, but it is redundant since the runtime type can be found by invoking `Object.GetType()`.

With this in mind, the method can be refactored to a non-generic version without sacrificing any function. The type of the instance argument changes to `object`, and `typeof(T)` on line 5 becomes `instance.GetType()`. The private generic methods in the class can be refactored similarly. The result is code that is simpler and more straightforward.

<script src="https://gist.github.com/joebuschmann/16035ad4645574c39519.js"></script>
