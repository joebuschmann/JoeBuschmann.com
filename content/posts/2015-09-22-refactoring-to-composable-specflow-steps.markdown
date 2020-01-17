---
layout: post
title: Refactoring to Composable SpecFlow Steps
date: '2015-09-22 12:15:32'
tags:
- net
- specflow
- c
---

I've seen some pretty bad SpecFlow code. Code that seems to violate every good practice out there. Poor reuse. Copy and paste everywhere. Test code is the hotel room of the software world. People are sloppier and more careless than they otherwise would be. I'm not sure why that is. Perhaps because tests are not seen as "real code". But as a testing code base grows from dozens to hundreds to even a thousand or more test cases, having well-factored composable SpecFlow steps becomes critical.

For this post, I'd like to share a strategy I use to solve a common use case. It doesn't take much effort and yields huge dividends over time.

#### Testing an Email Service

Most of my integration tests exercise web services, and many service operations require a request payload with setup spanning multiple Gherkin steps. Consider the example below. It validates a service responsible for sending marketing emails much like MailChimp's [Mandrill API](https://mandrillapp.com/api/docs/).

The Gherkin and bindings represent a basic test that sends a "Hello World" email message. The first three `Given` statements build out the email contents with a sender, recipients, and message body. Each one is responsible for updating the private field `_sendRequest` with different bits of information. It makes sense to break these out from the subsequent `When` and `Then` statements that send the message and verify the response. This honors the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) and makes for better reuse.

<script src="https://gist.github.com/joebuschmann/28643cae5e0530a02bbb.js"></script>

<script src="https://gist.github.com/joebuschmann/fecc6f449aa3d58aaf3b.js"></script>

#### SendRequestBuilder

The `SendRequestBuilder` class is the first step toward a better solution. It contains the three `Given` step bindings copied from `SendMailSteps`. They act on an internal `SendRequest` object exposed by the `SendRequestBuilder.Instance` property. SpecFlow can bind to the step definitions as before.

<script src="https://gist.github.com/joebuschmann/932892430de38e8e39b1.js"></script>

`SendMailSteps` cleans up nicely after introducing `SendRequestBuilder`. The constructor takes an instance via dependency injection, and `InvokeEndpoint()` grabs the request object before calling the service.

<script src="https://gist.github.com/joebuschmann/24defcbb362b1b1d9867.js"></script>

By moving these steps to a common builder, they can be shared among different feature classes.

#### Step Argument Transformations

While this is a move in the right direction, `SendRequestBuilder` could be improved further. Often SpecFlow steps need to be called directly from code in addition to binding to the Gherkin. You may notice the same request being created in several places and decide to condense those steps into one. For example, the code snippet below builds the "Hello World" request in one step rather than three.

<script src="https://gist.github.com/joebuschmann/c5f25addb0729409f4a9.js"></script>

Notice how awkward it is to create and populate a SpecFlow table in code. Tables work well for bindings, but manipulating them directly is messy. A good solution is to use [Step Argument Transformations](http://www.specflow.org/documentation/Step-Argument-Conversions/) to eliminate the table argument. Then the steps become much easier to call from code and can still be bound to tables.

<script src="https://gist.github.com/joebuschmann/d6bb47b534cd811a314d.js"></script>

Now the method `GivenTheHelloWorldEmailMessage()` uses concrete types instead of tables. Much better.

<script src="https://gist.github.com/joebuschmann/8fd9a129ad0795fab66d.js"></script>

#### Next Steps

As you can see, a little bit of encapsulation goes a long way. You could refactor this example further by moving the two `Then` steps into a shared validation class. In fact, a mature SpecFlow code base will often contain many builders and validation objects injected into lightweight steps classes.
