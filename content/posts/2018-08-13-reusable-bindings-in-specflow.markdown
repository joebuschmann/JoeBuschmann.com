---
layout: post
title: Reusable Bindings in SpecFlow
date: '2018-08-13 13:44:33'
tags:
- specflow
- c-sharp
- gherkin
- dotnet
---

As your application grows, your SpecFlow test suite needs to grow with it. Reusable bindings are essentional to prevent your code from becoming a bloated mess. Fortunately, the SpecFlow runtime has reusability in mind with a built-in inversion of control (IoC) framework and step argument transformations. With these features you can create reusable bindings that make test creation more productive.

### Problems with Inheritance

Before digging into IoC, I'll take a moment to cover why you shouldn't use inheritance in your bindings. I don't have any issues with inheritance in general, but with SpecFlow it can cause problems. The biggest issue is bindings in a base class can cause exceptions to be thrown at runtime. Check out the UML diagram below.

<img src="/images/inheritance-whiteboard.png" />

It shows a relationship between a base class `Service<TReq, TResp>` and two subclasses `ProductCatalogService` and `OrderService`. The base class contains common state shared among the subclasses as well as common methods for invoking a service endpoint during a test. It handles creating the proxy, serializing the request, and deserializing the response. It also has steps defined and shared among its subclasses.

At runtime, scenarios using these bindings will throw an exception. SpecFlow sees the step bindings in the base class as duplicate steps and doesn't know which one to invoke.

```
TechTalk.SpecFlow.BindingException : Ambiguous step definitions found
for step 'Given Invoke Service'
```

The exception can be avoided by pulling the steps out of the base class into a new one. Along with it will come the common methods for invoking a service endpoint. If this is all the class is doing, there's no point to keeping it around, and it should be removed.

Check out [Problems with placing step definitions to base classes](http://gasparnagy.com/2015/05/specflow-tips-problems-with-placing-step-definitions-to-base-classes/) by Gaspar Nagy for more detail on this issue.

Another problem with inheritance is base classes tend to become a dumping ground for state shared between the bindings. Over time, the shared state will have low cohesion and lead to violations of the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle). As you'll see, state classes should be kept small and shared using IoC.

### Reuse with the IoC Container

There's a better way to share state and code. SpecFlow ships with a [lightweight IOC container](https://github.com/gasparnagy/BoDi) that you can leverage to inject dependencies into bindings. Coding for dependency injection naturally leads to small classes with high cohesion and avoids the problems of inheritance.

Using the container is straightforward. Add a dependency to a binding's constructor, and the runtime will provide it. SpecFlow will automatically pick up public classes with a parameterless constructor or a constructor whose dependencies the runtime can resolve. Below is an example.

```csharp
[Binding]
public class Search
{
    private readonly IWebDriver _webDriver;
    private readonly ISearchProvider _searchProvider;

    public Search(IWebDriver webDriver, ISearchProvider searchProvider)
    {
        _webDriver = webDriver;
        _searchProvider = searchProvider;
    }

    // Snip
}
```

The `Search` binding has two dependencies: an instance of `IWebDriver` and `ISearchProvider`. It takes them as constructor parameters and saves them to a field variable.

What can you get from the container?

* Classes decorated with the Binding attribute
* Custom context classes
* Built-in context classes - e.g. ScenarioContext
* The container itself - IObjectContainer
* Dependencies explicitly registered with the container

Here are three examples of dependency injection (or DI) in SpecFlow.

#### Example 1: Retrieve the SpecFlow context objects

```csharp
private readonly ScenarioContext _scenarioContext;

public AddressSteps(ScenarioContext scenarioContext)
{
    _scenarioContext = scenarioContext;
}
```

You can retrieve metadata provided by the SpecFlow runtime which describes the current scenario. This metadata is contained in the `ScenarioContext` object provided by the framework.

#### Example 2: You can use DI to load the appropriate Selenium web driver

```csharp
[Binding]
public class BootstrapSelenium : IDisposable
{
    private readonly IObjectContainer _objectContainer;
    private IWebDriver _webDriver = null;

    public BootstrapSelenium(IObjectContainer objectContainer)
    {
        _objectContainer = objectContainer;
    }

    [BeforeScenario]
    public void LoadDriver()
    {
        _webDriver = BuildWebDriver();
        _objectContainer.RegisterInstanceAs(_webDriver, typeof (IWebDriver));
    }

    // Dispose will be called after scenario execution is complete
    public void Dispose()
    {
        _webDriver?.Quit();
    }
}
```

This example uses a BeforeScenario hook to load the appropriate Selenium web driver for web UI tests. The binding takes an instance of `IObjectContainer` in the constructor. The `BuildWebDriver` method creates an instance of `IWebDriver` using configuration or some other mechanism. Then the instance is registered with the container using `_objectContainer` and made available to bindings.

Note the binding also implements `IDisposable`. SpecFlow will invoke the `Dispose` method of any binding after the scenario is complete. In this case, it is the equivalent of applying the `[AfterScenario]` attribute.

#### Example 3: You can use tags to conditionally load dependencies

```gherkin
@json
Scenario: Invoke service with JSON payload
  Given a fully populated request object
  When I invoke the service
  Then a valid response should be returned
```

Tags are a Gherkin feature that are useful for annotating a scenario or feature and can drive conditional binding or custom behavior. Tags begin with the "@" character.

The preceding Gherkin uses the tag `@json` to indicate the scenario should be executed using JSON serialization. This scenario validates a service endpoint and needs to use the JSON data exchange format for the request and response data. The endpoint also accepts XML or FormUrlEncoded content.

```csharp
[BeforeScenario("xml")]
public void ConfigureXml()
{
    _objectContainer
        .RegisterTypeAs<XmlBodySerializer, IBodySerializer>();
}

[BeforeScenario("json")]
public void ConfigureJson()
{
    _objectContainer
        .RegisterTypeAs<JsonBodySerializer, IBodySerializer>();
}

[BeforeScenario("formurlencoded")]
public void ConfigureFormUrlEncoded()
{
    _objectContainer
        .RegisterTypeAs<FormUrlEncodingBodySerializer, IBodySerializer>();
}
```

The binding implements three `BeforeScenario` hooks for registering the correct serializer. The options are XML, JSON, and FormUrlEncoded. Only one serializer will be added to the container depending on which tag is used.

```csharp
[Binding]
public sealed class AddressServiceSteps
{
    private readonly IBodySerializer _bodySerializer;

    public ContentTypeSteps(IBodySerializer bodySerializer)
    {
        _bodySerializer = bodySerializer;
    }

    // Snip
}
```

Other bindings request the correct serializer by adding it to the constructor.

### What about the Steps class?

It's clear I prefer code reuse via DI over inheritance; however, there is an abstract class `TechTalk.SpecFlow.Steps` in the runtime library you can use as the base class for your bindings. You can check out [the source](https://github.com/techtalk/SpecFlow/blob/master/TechTalk.SpecFlow/Steps.cs) on GitHub. A partial definition is below.

```csharp
public abstract class Steps
{
    // Built-in context
    public ScenarioContext ScenarioContext { get; }
    public FeatureContext FeatureContext { get; }
    public TestThreadContext TestThreadContext { get; }
    public ScenarioStepContext StepContext { get; }

    // Call other binding steps
    public void Given(string step);
    public void Given(string step, Table tableArg);
    public void Given(string step, string multilineTextArg);
    public void Given(string step, string multilineTextArg, Table tableArg);

    // Snip
}
```

It's strange that a framework built around DI would provide a base class like `Steps`. It seems to exist for two reasons: 1) to provide access to the scenario, feature, and test metadata and 2) to expose the ability to invoke other Gherkin steps from a binding.

As we have seen, the test metadata like `ScenarioContext` can be obtained via DI without the fuss of a base class. This leaves the only other reason you would use `Steps` which is to invoke other steps from a binding. This is actually quite useful. Composite steps can be created to wrap multiple steps into one. Below are two Gherkin steps for building a customer and address

```gherkin
Given the customer
  | Salutation | First Name | Last Name |
  | Miss       | Liz        | Lemon     |
And the address
  | Line 1                | City     | State | Zipcode |
  | 30 Rockefeller Plaza  | New York | NY    | 10112   |
```

You may want to combine these into a single composite step. `GivenANewCustomerAndAddress` does just that using `Steps.Given`.

```csharp
[Given("a new customer and address")]
public void GivenANewCustomerAndAddress()
{
    Table customer = new Table("FirstName", "LastName", "Salutation");
    customer.AddRow("Miss" "Liz", "Lemon");

    Table address = new Table("Line 1", "City", "State", "Zipcode");
    address.AddRow("30 Rockefeller Plaza", "New York", "NY", "10112");

    Given("the customer", customer);
    Given("the address", address);
}
```

Using `Steps` to invoke other bindings is not as straightforward as it would seem. In this case, the two tables are created in code using strings for the column names and values. Even worse, the bindings are invoked using string values to identify the target step. The code is messy and fragile. It would be better if this method could be rewritten in a strongly-typed manner.

Fortunately, there is a another way.

### Reusable Bindings with StepArgumentTransformation

Another solution is to invoke the binding steps directly rather than through the `Steps` methods, but this still leaves the issue of table arguments. Building a table in code is not pleasant. You can work around this by pulling out table arguments and transforming them into strongly typed .NET objects using the `StepArgumentTranformation` attribute. With this approach, steps can be bound to Gherkin tables and called from composite bindings.

The binding step for creating a customer can be paired with a step argument transformation to move the table argument out and replace it with a strongly typed `Customer` instance.

```csharp
[StepArgumentTransformation("the customer")]
public Customer CreateCustomer(Table table)
{
    return table.CreateInstance<Customer>();
}

[Given(@"the customer")]
public void GivenTheCustomer(Customer customer)
{
    _customer = customer;
}
```

The same can be done for the address step.

```csharp
[StepArgumentTransformation("the address")]
public Address CreateAddress(Table table)
{
    return table.CreateInstance<Address>();
}

[Given(@"the address")]
public void GivenTheAddress(Address address)
{
    _address = address;
}
```

With the step argument transformations in place, the composite binding cleans up nicely with strongly typed objects. The customer and address bindings can be invoked safely from code, and they will still bind to the corresponding Gherkin steps.

```csharp
[Given("a new customer and address")]
public void GivenANewCustomerAndAddress()
{
    Customer customer = new Customer
    {
        Salutation = "Miss",
        FirstName = "Liz",
        LastName = "Lemon"
    };

    Address address = new Address
    {
        Line1 = "30 Rockefeller Plaza",
        City = "New York",
        State = "NY",
        Zipcode = "10112"
    };

    _otherBindings.GivenTheCustomer(customer);
    _otherBindings.GivenTheAddress(address);
}
```

#### Another Example

Step argument tranformations are also useful with other argument types. In the Gherkin, `When I remove the 5th product`, you may want to pass the number 5 as an integer value representing the ordinal position of the product. You could write a binding that captures the value "5th" and parses the argument.

```csharp
[When(@"I remove the (.*) product")]
public void RemoveProduct(string position)
{
    // Position comes in as 1st, 2nd, 3rd, etc.
    int index = ParsePosition(position);

    // Snip
}
```

Parsing arguments in a binding step is an anti-pattern, but you could fix this with a regular expression in the `When` attribute to pull out the numeric value. An even better approach is to create a step argument transformation to extract the index argument. It can even decrement the value by one for immediate consumption by zero-based arrays and lists. Now you have a reusable way of extracting ordinal positions from Gherkin steps.

```csharp
[StepArgumentTransformation(@"(\d+)(?:st|nd|rd|th)")]
public int GetIndex(int index)
{
    return index - 1;
}

[When(@"I remove the (.*) product")]
public void RemoveProduct(int index)
{
    _products.RemoveAt(index);
}

[When(@"I update the (.*) product to (.*)")]
public void UpdateProduct(int index, string productName)
{
    _products[index] = productName;
}
```

<hr />

The SpecFlow runtime provides tools for building reusable bindings including dependency injection via an IoC framework and step argument transformations. You should create small cohesive classes shared via DI and transform table arguments into concrete .NET objects. You should also avoid inheritance, the `Steps` class, and parsing arguments in your step definitions. With these techniques, your SpecFlow code will scale along with your application.