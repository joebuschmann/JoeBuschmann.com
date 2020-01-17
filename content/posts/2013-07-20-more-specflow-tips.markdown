---
layout: post
title: More SpecFlow Tips
date: '2013-07-20 11:21:08'
tags:
- c
- specflow
---

UPDATE (11/18/2016):

>I've written a number of posts since this one was published that cover advanced SpecFlow topics like [composable steps](http://joebuschmann.com/refactoring-to-composable-specflow-steps/), [tags done right](http://joebuschmann.com/specflow-tags-done-right/), [managing state](http://joebuschmann.com/strategies-for-managing-state-in-specflow/), [useful regex](http://joebuschmann.com/useful-regex-for-specflow-bindings/), etc. For a list of all my SpecFlow writing, you can click on the SpecFlow tag or go here: http://joebuschmann.com/tag/specflow. SpecFlow is a fantastic tool that's changed the way I develop software, and I hope it does the same for you. Happy testing!

<hr />

<p>My <a href="/some-specflow-tips">last post</a> covered three tips or best practices for <a href="http://www.specflow.org/specflownew/" target="_blank">SpecFlow</a> that covered manipulating the current ScenarioContext with extension methods, binding multiple Given/When/Then attributes, and using hooks. In this post IΓÇÖll be covering three more tips for working with SpecFlow tables.</p> <p>SpecFlow tables provide an easy way of specifying the data for a collection or an individual object. The table in the specification step is passed into the step definition as an instance of the <em>TechTalk.SpecFlow.Table</em> class.</p> <p>When specifying a collection of data, each table row represents an item in the collection. The table headers indicate to which property the column values should be assigned. The example below defines a list of items with each item having an <em>Name</em>&nbsp; and a <em>Price</em> property.</p>

<script src="https://gist.github.com/joebuschmann/d2bde7c2377d6317db12a4015ffa6877.js"></script>

<p>Another use for a table is to specify the values of properties for a single instance of an object. For example the FirstName, LastName, and Email properties of a customer object could be listed with a name/value pair for each property.</p>

<script src="https://gist.github.com/joebuschmann/83a5a9b279602d844d662f4e1f42fc14.js"></script>

<p>Generally, IΓÇÖve found defining tables in specification steps to be straightforward. The more tedious part is translating the table data in the step definitions.</p>
<h3>Avoid Explicitly Enumerating Table Rows</h3>
<p>I try to avoid enumerating table rows in code as much as possible. Fortunately, there are helper methods in the <em>TechTalk.SpecFlow.Assist.TableHelperExtensionMethods</em> class that make translating from a table instance to a collection or object much easier. IΓÇÖve found very few instances where these methods didnΓÇÖt give me what I needed.</p>
<p>The following step definitions explicitly enumerate a table to retrieve data. The first one builds out a list of items in a shopping cart and the second, a customer instance. They correspond to the step specifications in the previous section.</p>

<script src="https://gist.github.com/joebuschmann/19dd4baf4d4ac3add1632faf8abdf99b.js"></script>

<p>This code cleans up nicely with calls to <em>table.CreateSet()</em> and <em>table.CreateInstance()</em>. The number of lines drops to two per step.</p>

<script src="https://gist.github.com/joebuschmann/efdaa71bafb903f1c61069649a19b0f7.js"></script>

<h3>Convert Table Data to Anonymous Types for Ease of Use</h3>
<p>The extension methods in <em>TechTalk.SpecFlow.Assist.TableHelperExtensionMethods</em> are useful for simple one-to-one mappings between table values and class properties; however they cannot be used for more complex mappings which require manual intervention. LetΓÇÖs say the shopping cart in the previous examples is enhanced to include discounts as child items under the corresponding product. This makes the ShoppingCartItem class hierarchal and difficult to model in a flat table structure. A new column called ParentItem has been added to the specification to define the parent-child relationships, but it doesnΓÇÖt map easily to the new list of child items in the ShoppingCartItem class. This means <em>table.CreateSet()</em> canΓÇÖt be used.</p>

<script src="https://gist.github.com/joebuschmann/410764b3abe47d6193fbb0535a37df1b.js"></script>

<script src="https://gist.github.com/joebuschmann/d808736c319331a4f7935e1bf22c8a41.js"></script>

<p>With this new requirement, the step definition becomes:</p>

<script src="https://gist.github.com/joebuschmann/9134f2e079f0c7e4b1b02d5cfd935519.js"></script>

<p>This step definition works, but I donΓÇÖt like now the row indexer is used throughout the method. I think this makes the code less readable.</p>
<p>As an alternative, the table could be converted to a list of objects up front. Anonymous types let us avoid having to define a class for such a simple and transient use case, and the remaining code uses the anonymous type which keeps things neat.</p>

<script src="https://gist.github.com/joebuschmann/7b425ace0ff9a75f2e9c5e425108f993.js"></script>

<h3>Flatten Object Hierarchies When Doing Table Comparisons</h3>
<p>Now we have steps for reading in order items and saving them as a list of <em>ShoppingCartItem</em> instances in the <em>ScenarioContext</em>. Subsequent steps can manipulate the order which in turn updates the shopping cart. It needs to be validated at the end of the test to ensure correct behavior.</p>

<script src="https://gist.github.com/joebuschmann/412632d86f9601b026bc136e71cb7d5d.js"></script>

<p>The flat list of expected shopping cart items needs to be validated against the hierarchal shopping cart. The step definition below does just that, but it violates tip one which is to avoid explicitly enumerating table rows. It is also difficult to follow since it needs to iterate through the child shopping cart items.</p>

<script src="https://gist.github.com/joebuschmann/1156173b7d666f3e34d081a26c6a5293.js"></script>

<p>Ideally we would like to use the helper extension methods to do the table comparison. This is possible by first flattening out the shopping cart hierarchy into a list. The <em>Flatten()</em> extension method does the trick and is invoked right before <em>table.CompareToSet()</em>. Now the step is just two lines of code. Much better.</p>

<script src="https://gist.github.com/joebuschmann/3f8ba6fd14a1cc7f6a7815dcbe5ec21d.js"></script>

<p>ThatΓÇÖs all I have for now. For more information on tables and the table helper methods, check out the links below.</p>
<p><a href="http://specflow.org/documentation/SpecFlow-Assist-Helpers/">SpecFlow Assist Helpers</a></p>
<p><a href="http://specflow.org/documentation/">Full SpecFlow Documentation</a></p>