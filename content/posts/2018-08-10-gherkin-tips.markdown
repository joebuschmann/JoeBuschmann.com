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

```gherkin
Scenario: Build a customer
  Given the customer
    | Name           | Address                                   |
    | Miss Liz Lemon | 30 Rockefeller Plaza; New York; NY; 10112 |
```

A better approach is to create two steps: one for the customer and one for the address. Each discrete piece of data should have its own column in the table. The name is broken up into Salutation, First Name, and Last Name columns. The address step has columns for Line 1, City, State, and Zipcode.

```gherkin
Scenario: Build a customer
  Given the customer
    | Salutation | First Name | Last Name |
    | Miss       | Liz        | Lemon     |
  And the address
    | Line 1                | City     | State | Zipcode |
    | 30 Rockefeller Plaza  | New York | NY    | 10112   |
```

### Use natural language instead of tech speak

By design, Gherkin should use natural language accessible to business owners. Tech speak has a tendency to sneak in over time. This example contains text referring to an index of a list or array. It makes sense to engineers but may be confusing to the business.

```gherkin
When I remove the product at index 5
```

You can take advantage of regular expressions to refactor this step to use the more natural language, "the 5th product", and still extract the index value in the binding.

```gherkin
When I remove the 5th product
```

### Avoid numeric IDs

Many integration test suites will use a well known data set to drive the tests. These data may be in a file, database, or available via a web service. Over time test engineers will memorize the numeric identifiers for entities used in common scenarios and start to refer to the entity IDs rather than a descriptive name. Try to avoid using these IDs in Gherkin.

```gherkin
Scenario: Update price
  Given the product with ID 46
  When the price is updated to $199.00
  Then the price should be saved
```

Outside of engineering, no one knows what product 46 is. You can make this better by defining a map of descriptive names to IDs in a feature's background steps. The Gherkin uses the name while the steps map the incoming name to its ID. In this case, product 46 maps to the more descriptive identifier "iPhone 6".

```gherkin
Background:
  Given I have the following products
    | Id | Name       |
    | 45 | Galaxy S5  |
    | 46 | iPhone 6   |
    | 47 | Nokia Icon |
 
Scenario: Update price
  Given the product "iPhone 6"
  When the price is updated to $199.00
  Then the price should be saved
```

### Put rules and equations in the feature file description

The feature header and description is a good place to define the rules and equations under test. If you're validating an insurance quoting engine, you can describe its rules at the top and avoid cluttering the scenarios with details of the algorithm.

```gherkin
Feature: Insurance Quote
  Test the quoting engine.
```

An obvious and lazy description helps no one. You might as well leave it out. A good description describes the algorithm you're testing. In this case, the insurance quote engine starts with a baseline price and adjusts it up or down based on the insured's individual risk and their neighbor's risk.

```gherkin
Feature: Insurance Quote
  Quotes begin with a baseline price.
  It is adjusted by the insured individual's risk score.
  It is further adjusted by other scores from people
    living near the insured.
  quote = baseline *
    (individual adjustment + weight * (co-located adjustment - 1))
```

The following scenarios simply plug in the numbers and validate the final quote.

```gherkin
Scenario Outline: Calculate Quote
  Given a baseline price of $<baseline>
  And an individual adjustment of <indvAdjustment>
  And a co-located adjustment of <coLocAdjustment> and weight of <weight>
  When I calculate the final quote
  Then the amount should be $<amount>

Examples:
  | baseline | indvAdjustment | coLocAdjustment | weight | amount |
  | 50       | 0.90           | 0.95            | 0.50   | 43.75  |
  | 50       | 1.15           | 1.30            | 0.50   | 65.00  |
```

<hr />

There are many more tips for writing effective Gherkin, but I've found these to be the most unique and useful. For more tips, check out the links below.

* [15 Expert Tips for Using Cucumber](https://www.engineyard.com/blog/15-expert-tips-for-using-cucumber)
* [9 tips for improving Cucumber test readability](https://www.foreach.be/blog/9-tips-improving-cucumber-test-readability)
* [BDD 101: Writing Good Gherkin](https://automationpanda.com/2017/01/30/bdd-101-writing-good-gherkin/)