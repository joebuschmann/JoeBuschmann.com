---
layout: post
title: SpecFlow NUnit3 Generator Plugin
date: '2017-10-22 02:44:18'
tags:
- specflow
- c
---

I recently created a SpecFlow plugin to solve a peculiar problem with NUnit test code generation. The issue is SpecFlow will generate test code that doesn't compile when the .NET project containing the SpecFlow scenarios has a default namespace with the word NUnit. For example, if your project has the namespace `IntegrationTests.NUnit`, then you'll receive the following error when trying to build.

>The type or namespace name 'Framework' does not exist in the namespace 'IntegrationTests.NUnit' (are you missing an assembly reference?)
>
>Cannot resolve symbol 'Framework'

The conflict arises in the NUnit attributes in the test code. They are all qualified by the namespace `NUnit.Framework`, but because the project contains the default namespace `IntegrationTests.NUnit`, the compiler will expect the NUnit library types to be under `IntegrationTests.NUnit.Framework`. The work-around is either 1) to change the project's default namespace and update all of its types or 2) to qualify the NUnit attributes with C#'s `global` keyword. I recommend approach one when possible, but approach two is more fun and makes for a good blog post.

To implement approach two, I created a SpecFlow plugin to alter the behavior of the default NUnit3 code generator to qualify namespaces with `global`.

For example:

```
[NUnit.Framework.TestFixtureAttribute()]
```

becomes

```
[global::NUnit.Framework.TestFixtureAttribute()]
```

Now the compiler knows to resolve the attribute type starting at the global namespace instead of the project namespace.

### The Plugin Implementation

There are three types of SpecFlow plugins (Runtime, Generator, and Runtime-Generator). For this scenario, I needed to create a Generator plugin which is made of two parts. The first is an implementation of `IGeneratorPlugin` which allows you to alter the behavior of SpecFlow by updating the dependencies in its container. The snippet below replaces the implementation of `IUnitTestGeneratorProvider` with a new one that adds the global keyword.

<script src="https://gist.github.com/joebuschmann/34573c6f0f2fea71421db5fb61c79c9c.js"></script>

The second part is the new implemenation of `IUnitTestGeneratorProvider`. It augments the behavior of the default NUnit3 code generator to prefix each attribute's fully qualified type declaration with `global::`.

<script src="https://gist.github.com/joebuschmann/78b2861edce869ae1375896531b4a5c6.js"></script>

These two classes complete the plugin. You can compile them into an assembly and configure the SpecFlow Visual Studio extension to load it. For more details on creating and configuring plugins, check out the [SpecFlow documentation](https://specflow.org/documentation/Plugins/). Also, you can find [the full source](https://github.com/joebuschmann/specflow-plugin-nunit3-with-global) on GitHub.