---
layout: post
title: Getting Started with Selenium, SpecFlow, and .NET
date: '2017-05-30 19:54:01'
tags:
- net
- specflow
- selenium
---

Selenium is a library used to automate web browsers. It uses a common web driver interface, and each web browser, Chrome, Firefox, IE, has a corresponding implementation that takes advantage of the browser's native API.

I don't have much experience with browser automation. In fact, I try to avoid it as much as possible in favor of testing at the unit and service level. Tests running through the browser tend to be brittle due to the non-deterministic nature of web pages. Despite this, familiarity with Selenium is still valuable due to its popularity.

The following is a guide to get newcomers up and running with Selenium in .NET. It includes project setup, a simple scenario, and refactoring for more maintainable code.

#### NuGet Packages

Before we get into the details of Selenium, you'll need to create a new project in Visual Studio and add some dependencies from NuGet. Below are the NuGet packages I typically include.

1. NUnit (or any other unit testing framework supported by SpecFlow)
2. SpecFlow
3. Selenium.WebDriver
4. Selenium.WebDriver.ChromeDriver (or any other browser driver)
5. Selenium.Support (optional)
6. NUnit.ConsoleRunner (optional)

**Selenium.Support** is optional but highly recommended. It contains useful helper methods that complement the Selenium.WebDriver API. For example, the Selenium API is asynchronous and non-deterministic, and Selenium.Support's waiting methods make using it much easier.

NUnit.ConsoleRunner is also optional if you're using NUnit and need a test runner. Other options include using Resharper's test runner or NUnit's CLI.

#### Google Search

Now that the project is set up, the next step is to choose a simple example to demonstrate the basics of Selenium. I went with validating a Google search. The SpecFlow feature definition below includes Gherkin for validating a search for the keyword *kittens*.

<script src="https://gist.github.com/joebuschmann/53ab562884b47378fe577dc9a70a205a.js"></script>

Next I added a SpecFlow step definition file.

<script src="https://gist.github.com/joebuschmann/a919a84a75081008151ef7da71bbb428.js"></script>

#### IWebDriver

At this point the test should run and pass. Let's take a closer look at how Selenium works. The core of its architecture includes the `IWebDriver` interface and an implementation for each of the major browsers. In this example, I went with the Chrome driver; however, a more robust test suite would include the other browsers and the ability to switch between them.

```
private readonly IWebDriver _webDriver;
```

The web driver field is declared as `IWebDriver` and is assigned in the constructor.

```
_webDriver = new ChromeDriver();
```

It will be used by the test to launch the browser, navigate to a web page, and manipulate the elements on the page.

#### IWait

The `WebDriver.Support` library is technically optional, but the reality is its helper classes make Selenium much easier to use. The most useful of these helpers are the implementations of `IWait`.

```
private readonly IWait<IWebDriver> _defaultWait;
```

I included a default wait configuration in the step bindings with a timeout of five seconds and a polling interval of 100 milliseconds.

```
_defaultWait = new WebDriverWait(_webDriver, TimeSpan.FromSeconds(5))
{
    PollingInterval = TimeSpan.FromMilliseconds(100)
};
```

The `WebDriverWait` class is useful when you need to find dynamic HTML elements on a page like those created after an XHR request. You don't know when the elements will appear, so the test steps need to poll until they load.

```
IWebElement searchResultsHeader = _defaultWait.Until(d =>
{
    var results = d.FindElements(By.CssSelector("h2"));
    return results.FirstOrDefault(h => h.GetAttribute("innerText") == "Search Results");
});

Assert.IsNotNull(searchResultsHeader);

IWebElement resultsDiv =
    _defaultWait.Until(
        ExpectedConditions.ElementExists(By.CssSelector($"div[data-async-context=\"query:{_searchTerm}\"]")));

Assert.IsNotEmpty(resultsDiv.Text);
```

The validation step uses `_defaultWait.Until()` to find the search results header and the `<div />` element containing the results. The Until method repeatedly invokes the callback argument until a non-null value is returned or the timeout expires. `ExpectedConditions.ElementExists` and `By.CssSelector` are useful helpers that build out the `Func<IWebDriver, IWebElement>` argument.

#### Cleaning Up

Once the test scenario has completed, the browser and accompanying command window need to be shut down. This is implemented as an AfterScenario hook.

```
[AfterScenario]
public void CleanUp()
{
    _webDriver.Quit();
}
```

#### Refactoring for Composition

At this point, the test runs well as an isolated example, but it could be improved to make it more [composable](https://en.wikipedia.org/wiki/Composability). Design choices like setting the browser and default wait parameters within the class make it difficult to reuse. What if we wanted to do a test run with Firefox? Or change the timeout to 10 seconds for a particular environment? One option is to create a base class with subclasses for  each browser. Another is to take advantage of SpecFlow's DI framework to inject dependencies like the IWebDriver implementation.

I decided to go with DI over subclassing. I pulled the web driver setup and tear-down code out of the `GoogleSearch` class and into its own class called `BootstrapSelenium`. I also moved the default wait code. In addition I added some application settings to make the web driver type and wait timeout/poll interval configurable. As more tests are added, they will not need to configure their own web drivers.

Another advantage of this change is `GoogleSearch` becomes solely focused on providing the step implementations for the Gherkin, and the bootstrap class handles test setup and tear-down. Each class is simpler and more in line with the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

Below are the two refactored classes. You can find the [full source on GitHub](https://github.com/joebuschmann/selenium-demo).

<script src="https://gist.github.com/joebuschmann/4fffd1bd7934eeac402c66fdbdf8cde0.js"></script>

<script src="https://gist.github.com/joebuschmann/a4b81045917c5359dfb13d9403b6a202.js"></script>