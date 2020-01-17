---
layout: post
title: SpecFlow Tags Done Right
date: '2015-10-26 13:30:51'
tags:
- specflow
- net
- c
---

In a [previous post](http://joebuschmann.com/an-introduction-to-scoped-bindings-in-specflow/), I covered scoped bindings in SpecFlow and ended with an example of how *not* to use tags. In this post, I'll cover the "right way" and demonstrate how to avoid [coupling features to step definitions](https://github.com/cucumber/cucumber/wiki/Feature-Coupled-Step-Definitions-%28Antipattern%29). But first, a quick primer on tags.

#### What Are Tags?

Tags are used in Gherkin to mark features or scenarios. They begin with the `@` character in Gherkin, but in step definitions the `@` is removed.

Tags are used for:

1. Mapping to categories in the underlying unit test framework (not all frameworks support categories).
2. Creating scoped bindings in conjunction with the `Scope` attribute.
3. Determining which before and after hooks execute.
4. Custom logic in step bindings. Your code can inspect the `ScenarioContext.Current.ScenarioInfo.Tags` or `FeatureContext.Current.FeatureInfo.Tags` properties to determine which tags have been applied.

#### The Wrong Way

I ended my previous post with an example demonstrating how tags could be used, and I also warned of the tight coupling between the Gherkin and the steps. There's a better way, but first take a moment to review the tests below. Each one invokes a service with the request body encoded differently. There's a test for XML, JSON, and form-urlencoding. The method bound to the `When` step is determined by the tag applied to the scenario.

There are a couple of problems with this implementation. First, like I mentioned previously, these steps are coupled to the Gherkin, and it would be difficult to reuse them elsewhere. Second, the code for serializing the body is in the steps (excluded below for brevity's sake). Other steps will need to duplicate this code if they want to do any serialization.

<script src="https://gist.github.com/joebuschmann/9e9ae7dab81f27cfee61.js"></script>

<script src="https://gist.github.com/joebuschmann/fca3c06961a6dd0b7443.js"></script>

#### The Right Way

The first step toward a better solution is to pull out the serialization code and encapsulate it into three reusable components. Each component will also expose the appropriate content type header.

<script src="https://gist.github.com/joebuschmann/4eebf78beacfc757c65c.js"></script>

Now that the serialization code is encapsulated, it can be registered with the DI container in a `BeforeScenario` hook. The implementation is determined by which tag decorates the scenario.

<script src="https://gist.github.com/joebuschmann/24388a35ea1e3d72434c.js"></script>

The steps clean up nicely. The correct implementation of `IBodySerializer` is injected via the constructor, and the three methods to invoke the service collapse into one simple method. It uses the instance of `IBodySerializer` to serialize the body contents with the correct encoding and provide the value for the content type header.

<script src="https://gist.github.com/joebuschmann/c415d3597b1f0d529bc6.js"></script>

#### Wrapping It Up

That's a lot of code, but the idea is all step bindings work against the `IBodySerializer` interface. Its implementation is determined up front by a scenario tag in a hook and added to the container. Step methods don't care what's behind the interface and are decoupled from the implementation.

Another thing to note is this implementation favors composition over inheritance. I've seen similar problems solved using a base class plus subclasses scoped by tag. While this approach simplifies the step methods, it doesn't solve the lack of re-usability and coupling.