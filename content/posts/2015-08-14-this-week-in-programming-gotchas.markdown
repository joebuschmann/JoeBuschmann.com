---
layout: post
title: This Week in Programming Gotchas
date: '2015-08-14 12:08:00'
tags:
- net
- c
- nancy
- knockout-js
---

We all have those days where we spend hours trying to solve a seemingly simple problem only to smack ourselves in the head when we finally figure it out. Missing quotes, forgetting to flush a `StreamWriter`, etc. If only we could get back all the wasted time.

Well, this week I wrestled with more gotchas than usual. Below are the ones that burned most of my time.

###### Streams

Don't forget to flush a [StreamWriter](https://msdn.microsoft.com/en-us/library/system.io.streamwriter%28v=vs.110%29.aspx) when you're done. Otherwise, you'll be deep in the debugger trying to figure out why the target stream is empty.

```
// Don't forget to flush!
// Good advice for kids and .NET developers.
StreamWriter.Flush();
```

Oh, and while we're on the subject of Streams, after writing to a stream, you'll need to reset the stream's position pointer to 0 before reading from it. Some third party methods are smart enough to do this for you; others are not (I'm looking at you `HttpClient`).

```
// After writing to a stream, you'll need to manually reset the position
// before reading from it.
Stream.Position = 0;
```
###### Knockout.js

[Knockout.js](http://knockoutjs.com/) landed some good blows this week as I dug deeper into the library. In particular, it took hours and a number of searches to figure out how to bind `optionsText` and `optionsValue` in an HTML select element. Don't forget the single quotes or else Knockout will return a reference error.

>Unable to process binding "options: function (){return availableContentTypes }"
Message: display is not defined

```
    <!-- Use single quotes when binding a viewmodel property to optionsText and optionsValue. -->
    <p> <select data-bind="options: availableContentTypes,
                       optionsText: 'display',
                       optionsValue: 'value',
                       value: contentType"></select></p>
```

Frankly, I don't know why single quotes are necessary in this case, and I didn't bother to look it up. If anyone knows, please leave a comment.

###### Nancy

I stood up a website using [Nancy](http://nancyfx.org/) earlier in the week and had some static content to serve up. The easiest way to do this is to add to the static content conventions in a custom bootstrapper class. But when adding a static file to the root directory, you need to include the forward slash in both the requested file and content file.

```
    public class CustomBootstrapper : DefaultNancyBootstrapper
    {
        protected override void ConfigureConventions(NancyConventions conventions)
        {
            base.ConfigureConventions(conventions);
            conventions.StaticContentsConventions.AddDirectory("Scripts");

            // Include the forward slash in the file names.
            // If the file is part of a longer path, include the path.
            conventions.StaticContentsConventions.AddFile("/SendEmail.html", "/SendEmail.html");
        }
    }
```

###### It wasn't all bad

Despite the frustration, it really was a good week. Digging through these issues took me down some unexpected tangents. I learned a lot and was able to improve other areas of the codebase. Hopefully this post/rant prevents someone else from making the same mistakes.