---
layout: post
title: Useful Regex for SpecFlow Bindings
date: '2014-10-18 10:20:35'
tags:
- c
- specflow
---

Below is a list of useful regular expressions (regex) for annotating SpecFlow bindings. I'm treating this post as a reference and will be updating it with new items from time to time.

#### Singular or Plural

Support singular or plural wording.

<script src="https://gist.github.com/joebuschmann/4b35d619738733174332.js"></script>

<script src="https://gist.github.com/joebuschmann/aaacdf141e9b764bd6c6.js"></script>

#### Filter for Enum Values

If your binding takes an enumeration for an argument, you can limit the inputs from the Gherkin to just the valid values. In the snippets below, the possible values for the `SortOrder` argument are limited by the regex `(ascending|descending)` thus avoiding any runtime exeptions due to a bad value.

<script src="https://gist.github.com/joebuschmann/dff4197d91065e9008a8.js"></script>

<script src="https://gist.github.com/joebuschmann/316119805f84f5f0b97f.js"></script>

#### Step Argument Transformations

If the ascending/descending text feels a little awkward in the Gherkin, you can take the previous example one step further and add a step argument transformation to handle less technical wording. The regex in the step definition becomes `(.*)`, and the regex in the step argument transform becomes `from (A-Z|Z-A)`. The transform takes the more readable text "from A-Z" or "from Z-A" and converts it into a `SortOrder` value.

<script src="https://gist.github.com/joebuschmann/fedf72efd71d3b1f148d.js"></script>

#### Convert 1st, 2nd, 3rd, etc to Integer

Have you ever come across awkward Gherkin like this?
```
When I remove the item at index 10
```
As it's written, the index value 10 is easy to extract into an argument, but the wording isn't very human friendly. Instead, you can update the wording to indicate the ordinal position without sacrificing readability and use a regex to extract the integral value.

<script src="https://gist.github.com/joebuschmann/103821c6047f7109ddc1.js"></script>

<script src="https://gist.github.com/joebuschmann/a52cd96573ae4f21e240.js"></script>

#### Make "I" Optional

Repeating pronouns at the beginning of each step can get tedious. The previous example can be tweaked to remove the repetative "I"s.

<script src="https://gist.github.com/joebuschmann/3b802a916892d065693a.js"></script>

<script src="https://gist.github.com/joebuschmann/4883317c7d53cc0e80f8.js"></script>

