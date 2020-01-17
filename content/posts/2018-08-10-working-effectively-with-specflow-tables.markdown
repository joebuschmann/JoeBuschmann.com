---
layout: post
title: Working Effectively with SpecFlow Tables
date: '2018-08-10 21:41:47'
tags:
- specflow
- gherkin
- c
- net
---

The Gherkin DSL defines data tables as a way of passing a list of values to a step definition. Gherkin tables use the pipe character `|` to delimit column names and values. They're easy to read and understand by both business and technical people.

While they work great in Gherkin, tables don't translate well to strongly typed .NET languages. They are converted to an instance of the `Table` type in SpecFlow bindings. This data type is prone to errors because it is weakly typed (columns and values are strings) and requires iterating columns and rows to get at the values. A robust reusable library of SpecFlow bindings has to include strategies for working with tables effectively. Fortunately, there are helper libraries and patterns available to minimize the pain of manipulating table data.

### Vertical versus horizontal tables

Before digging in, a word about vertical and horizontal tables. Vertical tables have two columns. The first contains the field names, and the second contains the values. Vertical tables can only be mapped to a single .NET object, not lists or collections.

<script src="https://gist.github.com/joebuschmann/02829300cb545ecd1dfa9af615c26ad4.js"></script>

Horizontal tables have three or more columns. They are more flexible because they can be mapped to a list of objects. The first row defines the field names and each subsequent row holds the values for an item in the list.

<script src="https://gist.github.com/joebuschmann/47894d5023ef737efcde0d75130d8860.js"></script>

### Table values should be atomic

Table values should be as atomic as possible to simplify the .NET bindings comsuming them. If you find yourself parsing table values, that's a strong indication they're not atomic and can be broken down further.

The following Gherkin is a good example of what not to do. The first value is a full name including salutation. Names are normally divided into salutation, first name, and last name in code for storage and manipulation. The underlying binding will have to parse this value which is error prone. Same for the address. The different parts of the address (line1, city, state, and zip code) are delimited by a semi-colon. What if the person who created the Gherkin forgets the correct delimiter? Again this approach is error prone.

<script src="https://gist.github.com/joebuschmann/6b256eb93d81092ed75b874c1a9a4476.js"></script>

A better approach is to break up the name and address into atomic parts in the Gherkin. Each component is clearly defined. The name has separate columns for the salutation, first name, and last name. Similarly, the address is broken up into line1, city, state, and zip code. As we'll see in the next section, the .NET bindings simplify further with table helpers.

<script src="https://gist.github.com/joebuschmann/fe4147f06bddd8a419d81fbe0e8c2381.js"></script>

### Table helpers make working with tables easier

Like I mentioned earlier, the Gherkin language includes tables for passing complex data or lists of data to a step definition. They're easy to use in the Gherkin editor but are painful to work with in code. They're not strongly typed, and if there's a problem, you won't know until runtime.

A common pattern is to convert tables into strongly typed .NET objects in step definitions. In fact there are a number of helper extension methods in the SpecFlow runtime library for converting tables into objects and comparing tabular data to object data. These helpers can go a long way to clean up bindings.

<script src="https://gist.github.com/joebuschmann/5a245e10e6187fc1d8185a9e869f321a.js"></script>

In this example, the given step builds out an `Address` object by manually iterating the rows and columns of the incoming table. This code is prone to errors due to the lack of type safety, and not to mention it takes a lot of keystrokes. The `CreateInstance` extention method reduces it to one line.

<script src="https://gist.github.com/joebuschmann/f6d6297087922aea16d077af7e8a5b3f.js"></script>

In a similar way, SpecFlow has helper methods for comparing tables to objects.

<script src="https://gist.github.com/joebuschmann/9af894b5c721299fde4c748d4944c5e1.js"></script>

`ValidateAddress` compares each table value field by field. Again, you can collapse this code down to a single line.

<script src="https://gist.github.com/joebuschmann/92180ebe7834d67afd1fada42dc2bd54.js"></script>

#### TechTalk.SpecFlow.Assist

These helper methods can be found in the `TechTalk.SpecFlow.Assist` namespace. They are extension methods off of the `Table` data type.

- **CreateInstance** - creates a new object
- **FillInstance** - populates an existing object
- **CreateSet** - creates a list of objects
- **CompareToInstance** - compares table values to object properties
- **CompareToSet** - compares a table to a list of objects

Check out the [SpecFlow documentation](http://specflow.org/documentation/SpecFlow-Assist-Helpers/) for more details.

### Customize field mappings

SpecFlow will ignore whitespace and casing when matching table column names to object property names. Sometimes that isn't enough. An address object may have a property named "State", but for Canadian addresses, "Province" is the more appropriate term to use in the business domain. Same for "Zip Code" versus "Postal Code".

SpecFlow defines the `TableAliases` attribute for these situations. Using this attribute, you can provide alternate mappings between a table column name and an object property name. The runtime will include these mappings when you invoke the table helpers.

<script src="https://gist.github.com/joebuschmann/1c12785b58e305ee3c97e9b3c51c7235.js"></script>

The example above provides the alias "Province" for "State" and "Postal Code" for "Zip Code" among others. Now you can properly specifiy a Canadian address.

Also note that table aliases support regular expressions.

<script src="https://gist.github.com/joebuschmann/a4781bf01146c75389add9a894b7e36a.js"></script>

### Customize value mappings

Like field mappings, SpecFlow allows developers to customize how table values are mapped to an object property. The runtime handles primitive type conversions including Enums and Guids by default; however you may want to convert table values to a custom data type. You can do this with a custom value retriever and value comparer.

Let's say you want to update the address step with a new Location column. Location data consists of latitude and longitude values in parentheses and separated by a comma.

<script src="https://gist.github.com/joebuschmann/4966b3f1a1f83aaf9294240190d35b77.js"></script>

You want to map this value to a new property on the custom `Address` data type called Location. The property is of type `GeoLocation` which has properties for the latitude and longitude.

<script src="https://gist.github.com/joebuschmann/e50d1679ea8406dacac84eaaa8752795.js"></script>

Of course the SpecFlow runtime doesn't know about the GeoLocation type, but you can use custom implementations of `IValueRetriever` and `IValueComparer` to tell the runtime how to do the conversions. Value retrievers take a table value and convert it into an instance of a .NET type. Value comparers take a .NET type and compare it to a table value to determine equivalence. If the two values aren't equivalent, the runtime throws an exception, and the test fails.

#### IValueRetriever

`IValueRetriever` defines two methods. The first, `CanRetrieve`, returns a boolean value indicating if the value retriever can handle the specified property type. In this example, if the incoming property type is `GeoLocation`, then it returns true. The second method, `Retrieve`, performs the work of converting the string value from the table into an instance of the target type which, in this case, is `GeoLocation`.

<script src="https://gist.github.com/joebuschmann/4058287e99d16aa6068a947235d15a43.js"></script>

#### IValueComparer

`IValueComparer` is very similar in that it defines two methods, `CanCompare` and `Compare`. `CanCompare` is provided the property value from the target .NET object. In this case, if it is of type `GeoLocation`, then the method returns true indicating it can handle the comparison. The second method, `Compare`, is passed the same actual value and the expected string value from the table. The method does the work of comparing the two to determine equality. If it returns false, the test fails.

<script src="https://gist.github.com/joebuschmann/700b16a22e2b59c60d944d11498be8e3.js"></script>

#### Register Value Mappings

For the SpecFlow runtime to pick up custom value handlers, they have to be registered in a `BeforeTestRun` hook. `GeoLocationValueHandler` implements both interfaces and is passed to methods on `Service.Instance`. The runtime can now work with location values in the binding steps.

<script src="https://gist.github.com/joebuschmann/47eb7c3ad02d10876e26df6a5a1271f3.js"></script>

<hr />

The key to a robust reusable SpecFlow library is handling table data efficiently. By keeping values atomic, using the table helpers, and creating custom field and value mappings, your bindings will scale as the number of scenarios grows.