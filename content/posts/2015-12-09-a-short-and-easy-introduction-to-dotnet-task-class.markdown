---
layout: post
title: A Short and Easy Introduction to .NET's Task Class
date: '2015-12-09 22:09:29'
tags:
- dotnet
- c-sharp
- async
- parallel
- task
---

![Train Yard](/images/train-yard.png#c)

# Task.Run
You can use `Task.Run` to schedule a delegate to run on the thread pool. The method returns a new task, and if the work is complete, the result will be available via `Task.Result`. If not, `Task.Result` will block until it is complete.

```csharp
private void button_Click(object sender, EventArgs e)
{
    Task<DateTime> task = Task.Run(() => GetDateTime());
    lblDateTime.Text = task.Result.ToString();
}

private DateTime GetDateTime()
{
    // Simulate a blocking task.
    Thread.Sleep(2000);
    return DateTime.Now;
}
```

# Task.ContinueWith
You'll want to avoid accessing the `Task.Result` property because it will block until the result is ready. In the previous example, the UI will hang for about 2000 milliseconds before updating the label with the result. You can fix this with `Task.ContinueWith`.

```csharp
private void button_Click(object sender, EventArgs e)
{
    Task<DateTime> task = Task.Run(() => GetDateTime());
    task.ContinueWith(t => lblDateTime.Text = t.Result.ToString());
}

private DateTime GetDateTime()
{
    // Simulate a blocking task.
    Thread.Sleep(2000);
    return DateTime.Now;
}
```

In this case `Task.Result` will not block because `ContinueWith` runs after the first task is complete; however, this code will throw an exception.

> An exception of type 'System.InvalidOperationException' occurred in System.Windows.Forms.dll but was not handled in user code
>
>Additional information: Cross-thread operation not valid: Control 'lblDateTime' accessed from a thread other than the thread it was created on.

You can only update the UI from the thread that created the form. Remember `Control.Invoke` in WinForms? Fortunately, you can fix it by creating an instance of `TaskScheduler` from the UI thread's `SynchronizationContext` and using it when invoking `ContinueWith`.

```csharp
private void button_Click(object sender, EventArgs e)
{
    TaskScheduler uiThreadContext = TaskScheduler.FromCurrentSynchronizationContext();
    Task.Run(() => GetDateTime())
        .ContinueWith(dt => lblDateTime.Text = dt.Result.ToString(), uiThreadContext);
}
        
private DateTime GetDateTime()
{
    // Simulate a blocking task.
    Thread.Sleep(2000);
    return DateTime.Now;
}
```

# Awaiting Task.Run
If you're using C# 6.0, you can take advantage of async/await and await the task returned by `Task.Run`. The compiler will ensure the continuation runs on the originating thread, in this case the UI thread, so you don't have to create a `TaskScheduler`.

```csharp
private async void button_Click(object sender, EventArgs e)
{
    DateTime timeToDisplay = await Task.Run(() => GetDateTime());
    lblDateTime.Text = timeToDisplay.ToString();
}

private DateTime GetDateTime()
{
    // Simulate a blocking task.
    Thread.Sleep(2000);
    return DateTime.Now;
}
```

# Task.Delay
`Task.Delay` returns a task that will complete after a period of time. It can be awaited just like any other task.

```csharp
private async void button_Click(object sender, EventArgs e)
{
    await Task.Delay(2000);
    lblDateTime.Text = DateTime.Now.ToString();
}
```

# TaskCompletionSource
`TaskCompletionSource` serves as a wrapper around a task whose lifetime must be managed explicitly. No delegate is provided. It's useful for bridging the gap between older asynchronous patterns and newer code based on tasks.

For example, let's say `Task.Delay` didn't exist, and we wanted a more efficient alternative to `Thread.Sleep`. We can use `System.Threading.Timer`, but it would be nice to wrap it in a task. `TaskCompletionSource` can help us do just that.

```csharp
private async void button_Click(object sender, EventArgs e)
{
    DateTime timeToDisplay = await Sleep(2000);
    lblDateTime.Text = timeToDisplay.ToString();
}

private Task<DateTime> Sleep(int milliseconds)
{
    TaskCompletionSource<DateTime> tcs = new TaskCompletionSource<DateTime>();
    new Timer(o => tcs.SetResult(DateTime.Now), null, milliseconds, Timeout.Infinite);
    return tcs.Task;
}
```

A real world example can be found in the codebase for [Nancy FX](http://nancyfx.org), a lightweight web framework for .NET. Nancy can be hosted in a number of ways including in ASP.NET where [an implementation of IHttpAsyncHandler](https://github.com/NancyFx/Nancy/blob/master/src/Nancy.Hosting.Aspnet/NancyHttpRequestHandler.cs) invokes the task-based Nancy engine. `IHttpAsyncHandler` uses an older construct to implement asynchronous operations, and [Nancy uses TaskCompletionSource](https://github.com/NancyFx/Nancy/blob/master/src/Nancy.Hosting.Aspnet/NancyHandler.cs) to bridge the gap.

# Task.FromResult
`Task.FromResult` returns a completed task prepopulated with a result. No code is run on a background thread. In fact, no work is done besides creating the new task and immediately setting the result.

This method provides a way for asynchronous interfaces to have synchronous implementations. I've found it useful for creating mock classes for tests. The mock doesn't do any blocking work but still needs to return a task as a result. `Task.FromResult` is perfect for this scenario.

```csharp
public class ServiceMock : IServiceProxy
{
    public Task<ServiceResponse> SendAsync(ServiceRequest request)
    {
        var fakeResponse = new ServiceResponse(200, "application/text", "Hello World!");
        return Task.FromResult(fakeResponse);
    }
}
```

# Task.Run and ASP.NET
Using `Task.Run` for CPU-bound work makes sense in a desktop application. The long-running task executes on a background thread which keeps the UI thread free to respond to user input. The UI is more interactive leading to a better user experience.

But does it make sense in ASP.NET? In short, no. `Task.Run` will consume another thread from the ASP.NET process's thread pool resulting in two threads to service the same request.

There are inefficiencies even if you use async/await. The calling thread is recycled, but another one is needed to handle the task. That kind of churn in the thread pool is a poor use of resources.

For a more in depth explanation, see Stephen Cleary's [series on Task.Run etiquette](http://blog.stephencleary.com/2013/10/taskrun-etiquette-and-proper-usage.html). The entire series is worth a read, but part three, [Don't Use Task.Run in the Implementation](http://blog.stephencleary.com/2013/11/taskrun-etiquette-examples-dont-use.html), specifically addresses tasks in ASP.NET.
