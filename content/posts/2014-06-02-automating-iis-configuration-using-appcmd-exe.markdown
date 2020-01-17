---
layout: post
title: Automating IIS Configuration Using AppCmd.exe
date: '2014-06-02 14:04:48'
tags:
- iis
- appcmd
---

A couple weeks ago I wrote a script to completely automate the set up of IIS for some system tests at work. The tests require three IIS applications along with unique application pools, and the applications need to be segregated under a virtual directory to isolate them from other automated tests. This was done for a local set up for application developers.

This simple script took what could have been an error prone manual process and made it quick and mostly fool proof.

If you're not familiar with appcmd.exe, it is a command line utility for managing IIS 7+. Pretty much anything you can do with the IIS Manager GUI you can do with appcmd. The API is elegant and well thought out. Each invocation of appcmd requires two arguments, a command and an object type. The optional arguments include the object identifier and configuration parameters.

Say you want to get a list of all web applications:

```
C:\Windows\System32\inetsrv>appcmd list app
```

Or you want to add a web app:

```
C:\Windows\System32\inetsrv>appcmd add app /site.name:"Default Web Site" /path:/MyWebApp /physicalPath:C:\MyWebApp
```

Below is a complete example that demonstrates the power of appcmd. It is a batch script that builds out a website, virtual directory, application, and app pool.

<script src="https://gist.github.com/joebuschmann/2b87280535cdcc1d9b41.js"></script>

For a list of all available object types and commands, see [Getting Started with AppCmd.exe](http://www.iis.net/learn/get-started/getting-started-with-iis/getting-started-with-appcmdexe).