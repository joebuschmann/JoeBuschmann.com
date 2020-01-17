---
layout: post
title: Async/Await and Recursion
date: '2015-03-25 18:52:22'
tags:
- c
- await
- async
---

While using the new async/await keywords in C# 5.0 for the first time, I noticed an interesting aspect to how recursive methods behave when using `await`. For one method I was working on, Resharper notified me of a possible stack overflow exception with a "function is recursive on all paths" warning, but it didn't fail at runtime. Instead, it continued happily calling itself with no issues.

<script src="https://gist.github.com/joebuschmann/1c45eda3f0855d0ad682.js"></script>

What keeps it from failing is the use of the `await` keyword when calling `DoWorkAsync()`. Awaiting a method call has the effect of splitting the calling method in two. The first part executes and returns immediately when the awaitable method is invoked. The second part runs when control returns to the calling thread. In the example above, the deepest the call stack gets is two levels.

First recursive call:

![Stack Trace After Recursive Call](http://media.joebuschmann.com/async_await_stack_trace1.png)

After awaiting:

![Stack Trace After Awaiting](http://media.joebuschmann.com/async_await_stack_trace2.png)

In this case, Resharper was correct to report that the method was recursive in all paths, but the use of the await keyword prevents a stack overflow exception.