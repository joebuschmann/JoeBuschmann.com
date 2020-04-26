---
layout: post
title: Working Effectively with SpecFlow Tables
date: '2018-08-10 21:41:47'
tags:
- specflow
- gherkin
- c-sharp
- dotnet
---

The Gherkin DSL defines data tables as a way of passing a list of values to a step definition. Gherkin tables use the pipe character `|` to delimit column names and values. They're easy to read and understand by both business and technical people.

While they work great in Gherkin, tables don't translate well to strongly typed .NET languages. They are converted to an instance of the `Table` type in SpecFlow bindings. This data type is prone to errors because it is weakly typed (columns and values are strings) and requires iterating columns and rows to get at the values. A robust reusable library of SpecFlow bindings has to include strategies for working with tables effectively. Fortunately, there are helper libraries and patterns available to minimize the pain of manipulating table data.

### Vertical versus horizontal tables

Before digging in, a word about vertical and horizontal tables. Vertical tables have two columns. The first contains the field names, and the second contains the values. Vertical tables can only be mapped to a single .NET object, not lists or collections.

```gherkin
# A vertical table

| Field   | Value                |
| Line 1  | 30 Rockefeller Plaza |
| City    | New York             |
| State   | NY                   |
| Zipcode | 10112                |
```

Horizontal tables have three or more columns. They are more flexible because they can be mapped to a list of objects. The first row defines the field names and each subsequent row holds the values for an item in the list.

```gherkin
# A horizontal table

| Line 1                | City     | State | Zipcode |
| 30 Rockefeller Plaza  | New York | NY    | 10112   |
| 311 South Wacker Dr   | Chicago  | IL    | 60606   |
```

### Table values should be atomic

Table values should be as atomic as possible to simplify the .NET bindings comsuming them. If you find yourself parsing table values, that's a strong indication they're not atomic and can be broken down further.

The following Gherkin is a good example of what not to do. The first value is a full name including salutation. Names are normally divided into salutation, first name, and last name in code for storage and manipulation. The underlying binding will have to parse this value which is error prone. Same for the address. The different parts of the address (line1, city, state, and zip code) are delimited by a semi-colon. What if the person who created the Gherkin forgets the correct delimiter? Again this approach is error prone.

```gherkin
Scenario: Build a customer
  Given the customer
    | Name           | Address                                   |
    | Miss Liz Lemon | 30 Rockefeller Plaza; New York; NY; 10112 |
```

A better approach is to break up the name and address into atomic parts in the Gherkin. Each component is clearly defined. The name has separate columns for the salutation, first name, and last name. Similarly, the address is broken up into line1, city, state, and zip code. As we'll see in the next section, the .NET bindings simplify further with table helpers.

```gherkin
Scenario: Build a customer
  Given the customer
    | Salutation | First Name | Last Name |
    | Miss       | Liz        | Lemon     |
  And the address
    | Line 1                | City     | State | Zipcode |
    | 30 Rockefeller Plaza  | New York | NY    | 10112   |
```

### Table helpers make working with tables easier

Like I mentioned earlier, the Gherkin language includes tables for passing complex data or lists of data to a step definition. They're easy to use in the Gherkin editor but are painful to work with in code. They're not strongly typed, and if there's a problem, you won't know until runtime.

A common pattern is to convert tables into strongly typed .NET objects in step definitions. In fact there are a number of helper extension methods in the SpecFlow runtime library for converting tables into objects and comparing tabular data to object data. These helpers can go a long way to clean up bindings.

```csharp
[Given(@"the following address")]
public void GivenTheFollowingAddress(Table table)
{
    Address address = new Address();

    address.Line1 = table.Rows[0]["Line 1"];
    address.Line2 = table.Rows[0]["Line 2"];
    address.City = table.Rows[0]["City"];
    address.State = table.Rows[0]["State"];
    address.Zipcode = table.Rows[0]["Zipcode"];
}
```

In this example, the given step builds out an `Address` object by manually iterating the rows and columns of the incoming table. This code is prone to errors due to the lack of type safety, and not to mention it takes a lot of keystrokes. The `CreateInstance` extension method reduces it to one line.

```csharp
[Given(@"the following address")]
public void GivenTheFollowingAddress(Table table)
{
    Address address = table.CreateInstance<Address>();
}
```

In a similar way, SpecFlow has helper methods for comparing tables to objects.

```csharp
[Then(@"the address is")]
public void ValidateAddress(Table table)
{
    Assert.AreEqual(table.Rows[0]["Line 1"], _address.Line1);
    Assert.AreEqual(table.Rows[0]["Line 2"], _address.Line2);
    Assert.AreEqual(table.Rows[0]["City"], _address.City);
    Assert.AreEqual(table.Rows[0]["State"], _address.State);
    Assert.AreEqual(table.Rows[0]["Zipcode"], _address.Zipcode);
}
```

`ValidateAddress` compares each table value field by field. Again, you can collapse this code down to a single line.

```csharp
[Then(@"the address is")]
public void ValidateAddress(Table table)
{
    table.CompareToInstance(_address);
}
```

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

```csharp
public class Address
{
    [TableAliases("Street")]
    public string Line1 { get; set; }

    [TableAliases("Township", "Village", "Municipality")]
    public string City { get; set; }

    [TableAliases("Province")]
    public string State { get; set; }

    [TableAliases("Zip", "Zip\\s*Code", "Postal\\s*Code")]
    public string Zipcode { get; set; }
}
```

The example above provides the alias "Province" for "State" and "Postal Code" for "Zip Code" among others. Note that table aliases support regular expressions.

Now you can properly specifiy a Canadian address.

```gherkin
Given the address
  | Street          | Village | Province | Postal Code |
  | 110 Prairie Way | Elkhorn | Manitoba | P5A 0A4     |
```

### Customize value mappings

Like field mappings, SpecFlow allows developers to customize how table values are mapped to an object property. The runtime handles primitive type conversions including Enums and Guids by default; however you may want to convert table values to a custom data type. You can do this with a custom value retriever and value comparer.

Let's say you want to update the address step with a new Location column. Location data consists of latitude and longitude values in parentheses and separated by a comma.

```gherkin

Given the address
  | Street          | Village | Province | Postal Code | Location |
  | 110 Prairie Way | Elkhorn | Manitoba | P5A 0A4     | (42, 88) |
```

You want to map this value to a new property on the custom `Address` data type called Location. The property is of type `GeoLocation` which has properties for the latitude and longitude.

```csharp
public class Address
{
    [TableAliases("Street")]
    public string Line1 { get; set; }

    [TableAliases("Township", "Village")]
    public string City { get; set; }

    [TableAliases("Province")]
    public string State { get; set; }

    [TableAliases("Zip", "Zip\\s*Code", "Postal\\s*Code")]
    public string Zipcode { get; set; }

    public GeoLocation Location { get; set; }
}
```

Of course the SpecFlow runtime doesn't know about the GeoLocation type, but you can use custom implementations of `IValueRetriever` and `IValueComparer` to tell the runtime how to do the conversions. Value retrievers take a table value and convert it into an instance of a .NET type. Value comparers take a .NET type and compare it to a table value to determine equivalence. If the two values aren't equivalent, the runtime throws an exception, and the test fails.

#### IValueRetriever

`IValueRetriever` defines two methods. The first, `CanRetrieve`, returns a boolean value indicating if the value retriever can handle the specified property type. In this example, if the incoming property type is `GeoLocation`, then it returns true. The second method, `Retrieve`, performs the work of converting the string value from the table into an instance of the target type which, in this case, is `GeoLocation`.

```csharp
public bool CanRetrieve(KeyValuePair<string, string> keyValuePair,
    Type targetType, Type propertyType)
{
    return propertyType == typeof(GeoLocation);
}

public object Retrieve(KeyValuePair<string, string> keyValuePair,
    Type targetType, Type propertyType)
{
    string coordinates = keyValuePair.Value;
    GeoLocation location;

    if (TryGetLocation(coordinates, out location))
        return location;

    throw new Exception(
        $"Unable to parse the location coordinates {coordinates}.");
}
```

#### IValueComparer

`IValueComparer` is very similar in that it defines two methods, `CanCompare` and `Compare`. `CanCompare` is provided the property value from the target .NET object. In this case, if it is of type `GeoLocation`, then the method returns true indicating it can handle the comparison. The second method, `Compare`, is passed the same actual value and the expected string value from the table. The method does the work of comparing the two to determine equality. If it returns false, the test fails.

```csharp
public bool CanCompare(object actualValue)
{
    return actualValue is GeoLocation;
}

public bool Compare(string expectedValue, object actualValue)
{
    GeoLocation expectedLocation;

    if (TryGetLocation(expectedValue, out expectedLocation))
        return expectedLocation.Equals(actualValue);

    return false;
}
```

#### Register Value Mappings

For the SpecFlow runtime to pick up custom value handlers, they have to be registered in a `BeforeTestRun` hook. `GeoLocationValueHandler` implements both interfaces and is passed to methods on `Service.Instance`. The runtime can now work with location values in the binding steps.

```csharp
[BeforeTestRun]
public static void RegisterValueMappings()
{
    var geoLocationValueHandler = new GeoLocationValueHandler();
    Service.Instance.RegisterValueRetriever(geoLocationValueHandler);
    Service.Instance.RegisterValueComparer(geoLocationValueHandler);
}
```

<hr />

The key to a robust reusable SpecFlow library is handling table data efficiently. By keeping values atomic, using the table helpers, and creating custom field and value mappings, your bindings will scale as the number of scenarios grows.