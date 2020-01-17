---
layout: post
title: A Short and Easy Introduction to .NET's Task Class
date: '2015-12-09 22:09:29'
tags:
- net
- c
- async
- parallel
- task
---

###### Task.Run
You can use `Task.Run` to schedule a delegate to run on the thread pool. The method returns a new task, and if the work is complete, the result will be available via `Task.Result`. If not, `Task.Result` will block until it is complete.

<script src="https://gist.github.com/joebuschmann/19eedb7e0ba5d4479980.js"></script>

###### Task.ContinueWith
You'll want to avoid accessing the `Task.Result` property because it will block until the result is ready. In the previous example, the UI will hang for about 2000 milliseconds before updating the label with the result. You can fix this with `Task.ContinueWith`.

<script src="https://gist.github.com/joebuschmann/be4f90c6ed40a3c35b84.js"></script>

In this case `Task.Result` will not block because `ContinueWith` runs after the first task is complete; however, this code will throw an exception.

> An exception of type 'System.InvalidOperationException' occurred in System.Windows.Forms.dll but was not handled in user code
>
>Additional information: Cross-thread operation not valid: Control 'lblDateTime' accessed from a thread other than the thread it was created on.

You can only update the UI from the thread that created the form. Remember `Control.Invoke` in WinForms? Fortunately, you can fix it by creating an instance of `TaskScheduler` from the UI thread's `SynchronizationContext` and using it when invoking `ContinueWith`.

<script src="https://gist.github.com/joebuschmann/dfaacb3458326dbce229.js"></script>

###### Awaiting Task.Run
If you're using C# 6.0, you can take advantage of async/await and await the task returned by `Task.Run`. The compiler will ensure the continuation runs on the originating thread, in this case the UI thread, so you don't have to create a `TaskScheduler`.

<script src="https://gist.github.com/joebuschmann/25ddcdb32fc187ebd389.js"></script>

###### Task.Delay
`Task.Delay` returns a task that will complete after a period of time. It can be awaited just like any other task.

<script src="https://gist.github.com/joebuschmann/265f093156822962f4d6.js"></script>

###### TaskCompletionSource
`TaskCompletionSource` serves as a wrapper around a task whose lifetime must be managed explicitly. No delegate is provided. It's useful for bridging the gap between older asynchronous patterns and newer code based on tasks.

For example, let's say `Task.Delay` didn't exist, and we wanted a more efficient alternative to `Thread.Sleep`. We can use `System.Threading.Timer`, but it would be nice to wrap it in a task. `TaskCompletionSource` can help us do just that.

<script src="https://gist.github.com/joebuschmann/909e3d157b705002e022.js"></script>

A real world example can be found in the codebase for [Nancy FX](http://nancyfx.org), a lightweight web framework for .NET. Nancy can be hosted in a number of ways including in ASP.NET where [an implementation of IHttpAsyncHandler](https://github.com/NancyFx/Nancy/blob/master/src/Nancy.Hosting.Aspnet/NancyHttpRequestHandler.cs) invokes the task-based Nancy engine. `IHttpAsyncHandler` uses an older construct to implement asynchronous operations, and [Nancy uses TaskCompletionSource](https://github.com/NancyFx/Nancy/blob/master/src/Nancy.Hosting.Aspnet/NancyHandler.cs) to bridge the gap.

###### Task.FromResult
`Task.FromResult` returns a completed task prepopulated with a result. No code is run on a background thread. In fact, no work is done besides creating the new task and immediately setting the result.

This method provides a way for asynchronous interfaces to have synchronous implementations. I've found it useful for creating mock classes for tests. The mock doesn't do any blocking work but still needs to return a task as a result. `Task.FromResult` is perfect for this scenario.

<script src="https://gist.github.com/joebuschmann/b1000e857675d3f764f9.js"></script>

###### Task.Run and ASP.NET
Using `Task.Run` for CPU-bound work makes sense in a desktop application. The long-running task executes on a background thread which keeps the UI thread free to respond to user input. The UI is more interactive leading to a better user experience.

But does it make sense in ASP.NET? In short, no. `Task.Run` will consume another thread from the ASP.NET process's thread pool resulting in two threads to service the same request.

There are inefficiencies even if you use async/await. The calling thread is recycled, but another one is needed to handle the task. That kind of churn in the thread pool is a poor use of resources.

For a more in depth explanation, see Stephen Cleary's [series on Task.Run etiquette](http://blog.stephencleary.com/2013/10/taskrun-etiquette-and-proper-usage.html). The entire series is worth a read, but part three, [Don't Use Task.Run in the Implementation](http://blog.stephencleary.com/2013/11/taskrun-etiquette-examples-dont-use.html), specifically addresses tasks in ASP.NET.
