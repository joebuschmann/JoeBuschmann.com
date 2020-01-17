---
layout: post
title: 'SpecFlow Nested Tables: A Bad Idea'
date: '2013-10-22 07:45:06'
tags:
- specflow
---

<p>IΓÇÖve been using <a href="http://www.specflow.org/specflownew/" target="_blank">SpecFlow</a> to write behavior specifications for just under a year, and one question that comes up is whether or not nested tables are supported for creating complex or hierarchal types. Other folks have discussed this at <a href="http://stackoverflow.com/questions/5788964/specflow-and-complex-objects">Stack Overflow</a> and <a href="https://groups.google.com/forum/#!topic/specflow/9VLN7dTz7Kk">Google Groups</a>. Nested tables arenΓÇÖt currently supported, and thatΓÇÖs probably a good thing.</p> <p>The intent of the Cucumber syntax is to express business requirements in the language of the business. Nested tables donΓÇÖt add value from the business side because the nesting tends to make the step less readable and serves the developer more than the business. Anything that makes the steps less readable <a href="http://www.elabs.se/blog/15-you-re-cuking-it-wrong">should be avoided</a>.</p> <p>Below are a couple of examples of what nested tables might look like if they were supported. The steps populate a Customer type with the simple properties FirstName and LastName, and the nested table populates ContactInformation a collection of phone numbers represented by a complex type as well.</p> <p>The first example uses a double pipe delimiter to indicate the contact information. The second uses an indicator of some sort to point to the nested table. Both examples are difficult to read, and theyΓÇÖre fairly simple compared to some of the complex types found in real systems.</p><pre class="csharpcode"><p># Example 1: Embedded inside the contact info cell</p><p>Given the following customer</p><p>  | First Name | Last Name | Contact Information                |
  | Gary       | Smith     | || Contact Type || Number       || |
  |            |           | || HomePhone    || 513 555-1111 || |
  |            |           | || CellPhone    || 513 555-2222 || |
</p><p>&nbsp;</p><p>&nbsp;</p><p># Example 2: Contact info appears after the primary table specification</p><p>Given the following customer
  | First Name | Last Name | Contact Information |
  | Gary       | Smith     | {nested}            |
    | Contact Type | Number       |
    | HomePhone    | 513 555-1111 |
    | CellPhone    | 513 555-2222 |</p></pre>
<p>With that said, it would still be nice to be able to build out a complex object using tables. Consider the following specification that builds out the same complex customer object.</p><pre class="csharpcode">Given the following customer
    | First Name | Last Name |
    | Gary       | Smith     |
And the following contact information
    | Customer   | Contact Type | Number       |
    | Gary Smith | HomePhone    | 513 555-1111 |
    | Gary Smith | CellPhone    | 513 555-2222 |</pre>This is much more readable than nested tables.&nbsp; Even with more data it remains easy to follow.<pre class="csharpcode">Given the following customer
    | First Name | Last Name |
    | Gary       | Smith     |
    | Kim        | Green     |
And the following contact information
    | Customer   | Contact Type | Number       |
    | Gary Smith | HomePhone    | 513 555-1111 |
    | Gary Smith | CellPhone    | 513 555-2222 |
    | Kim Green  | CellPhone    | 513 555-3333 |
    | Kim Green  | WorkPhone    | 513 555-4444 |
And the following addresses
    | Customer   | Address Type | Street          | City       | State |
    | Gary Smith | Home         | 111 Main Street | Cincinnati | OH    |
    | Gary Smith | Work         | 111 Poplar Road | Cincinnati | OH    |
    | Kim Green  | Home         | 222 Main Street | Cincinnati | OH    |
    | Kim Green  | Work         | 333 Poplar Road | Cincinnati | OH    |</pre>
<p>Now imagine what this would look like with nested tables. No matter how elegant the proposed syntax, it would be very difficult for non-technical people to follow.</p>