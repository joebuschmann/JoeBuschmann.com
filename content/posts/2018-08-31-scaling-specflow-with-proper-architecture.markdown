---
layout: post
title: Scaling SpecFlow With Proper Architecture
date: '2018-08-31 01:07:38'
tags:
- specflow
- gherkin
- c
- net
---

> In July of 2018, I gave a talk at [KCDC](http://www.kcdc.info/) titled *SpecFlow: Moving Beyond the Basics*. Afterward, I changed the title to *Scaling SpecFlow* to more accurately reflect the topic. You can find the updated slide deck at https://joebuschmann.github.io/scaling-specflow/.
> 
> Over the next few weeks, I created a series of posts, one for each section of the talk, for those of you who perfer blog posts to slide decks. This page aggregates these pages in one place and takes you through each one in order.

## Abstract

So you've written some tests using SpecFlow, and you're feeling good about test coverage. But as the code base gets larger, some issues are starting to emerge. Problems like awkward Gherkin, duplicate code, lack of reusable components, tedious table manipulation, and unused or missing bindings. Like any code base, your test code will accumulate tech debt over time. What can you do to fix these issues?

## Scaling SpecFlow

* [Let's Start with the Basics](https://joebuschmann.com/specflow-basics/)
* [Gherkin Tips](https://joebuschmann.com/gherkin-tips/)
* [Taming Gherkin Tables](https://joebuschmann.com/working-effectively-with-specflow-tables/)
* [Reusable Bindings](https://joebuschmann.com/reusable-bindings-in-specflow/)
* [Running Scenarios in Parallel](https://joebuschmann.com/running-specflow-scenarios-in-parallel/)
* [The Step Definition Report](https://joebuschmann.com/specflow-step-definition-report/)