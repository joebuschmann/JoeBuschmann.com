---
layout: post
title: TaskCompletionSource - Bridging the Gap Between Old and New
date: '2016-08-15 13:28:32'
tags:
- task
- c
- net
- await
- async
---

In the latest versions of the .NET Framework, asynchronous work is represented by the **Task** class. A task is similar to a future or promise in other languages. You can create one in many ways the most common being `Task.Run()`. The result of a task is exposed by the `Task.Result` property. If the work is complete, then the property immediately returns a value; otherwise, it blocks until the operation is finished.

#### Older Patterns

Tasks are a powerful addition to the .NET Framework especially when used to create [awaitables](http://blog.stephencleary.com/2012/02/async-and-await.html); however, there is still plenty of code out there using older asynchronous patterns. A good example is the ASP.NET interface **IHttpAsyncHandler**.

<script src="https://gist.github.com/joebuschmann/9a7f1b648185e915603fbd2f778e5b6a.js"></script>

IHttpAsyncHandler predates tasks and is used for implementing asynchronous web request handlers for ASP.NET websites. Note the **IAsyncResult** interface along with the Begin/End methods. This is a variation of a common pattern in early versions of .NET code.

#### TaskCompletionSource

Mixing awaitables and an older asynchronous API can be challenging. In these scenarios we need to create an awaitable that wraps the API. Calling code can then use the async/await keywords.

This is where the **TaskCompletionSource** class can help. It serves as a wrapper around a Task instance and exposes methods to manipulate the task explicitly. Using TaskCompletionSource, you can set a result or exception on the task or cancel it. It provides an nice way to bridge the gap between older async code and newer code using awaitables.

#### Asynchronous Calls with IWSTrustChannelContract

An example of an older asynchronous API is the interface `IWSTrustChannelContract`. It defines a contract for interacting with a security token service. The synchronous method `Issue()` takes a token request object and returns a security token or raises an exception for an invalid request. The asynchronous equivalent is the method pair `BeginIssue()` and `EndIssue()` also defined on the interface. BeginIssue takes three arguments, the request object, a callback, and a state object, and kicks off the work. EndIssue is invoked by the callback to retrieve the results.

<script src="https://gist.github.com/joebuschmann/7cf72676354e17b7318ad120acf898e1.js"></script>

I won't go into the details behind issuing tokens. Instead I'll focus on how TaskCompletionSource can be used to wrap the BeginIssue/EndIssue methods with a task. The example below does just that. The method **IssueAsync** takes four arguments and returns an instance of `Task<SecurityToken>`. The details of invoking BeginIssue and EndIssue are hidden from the caller.

<script src="https://gist.github.com/joebuschmann/d8388de7a15009fd3e907d8894abeb37.js"></script>

The first line initializes a new instance of `TaskCompletionSource`. It will be used later when invoking the Begin/End methods.

```
TaskCompletionSource<SecurityToken> tcs = new TaskCompletionSource<SecurityToken>();
```

The last line of the method returns the wrapped Task instance.

```
return tcs.Task;
```

All the interesting parts are in between. There are a few lines to set up an instance of `IWSTrustChannelContract`, but you can skim through those until you get to `channel.BeginIssue()`. This is where TaskCompletionSource comes into play. BeginIssue is passed the request and a callback to be invoked when the asynchronous work is complete. The state argument isn't relevant in this case so null will suffice.

```
channel.BeginIssue(requestSecurityToken, asyncResult =>
{
    try
    {
        RequestSecurityTokenResponse requestSecurityTokenResponse;
        SecurityToken securityToken = channel.EndIssue(asyncResult, out requestSecurityTokenResponse);
        tcs.SetResult(securityToken);
    }
    catch (Exception e)
    {
        tcs.SetException(e);
    }
}, null);
```

Inside the callback lambda expression, the result is obtained by invoking EndIssue. Then the TaskCompletionSource instance is used either to set the result if all goes well, `tcs.SetResult(securityToken)`, or set an exception `tcs.SetException(e)` if something goes awry. These methods manipulate the wrapped Task instance directly. This is the same instance returned asynchronously by the **IssueAsync** method.

Now we have a method that exposes the older API as an awaitable. Callers can use the await keyword to invoke it.

```
SecurityToken securityToken = await IssueAsync(stsAddress, appliesToAddress, httpBinding, clientCredentials);
```

#### Event-based APIs

Other uses for TaskCompletionSource include exposing an event-based API as a Task. Below is a contrived yet simple example of a sleep method. It wraps a timer event as a Task.

<script src="https://gist.github.com/joebuschmann/909e3d157b705002e022.js"></script>

#### Wrapping Up

TaskCompletionSource is useful for bridging the gap between a task-based API and a non-task API. In fact, it is used internally throughout the TPL library.

For further reading, check out [Creating Tasks](http://blog.stephencleary.com/2012/02/creating-tasks.html) by Stephen Cleary and [The Nature of TaskCompletionSource<TResult>](https://blogs.msdn.microsoft.com/pfxteam/2009/06/02/the-nature-of-taskcompletionsourcetresult/) in the Microsoft TPL documentation.