---
layout: post
title: Tidying Up Code with C#'s Using Alias Directives
date: '2015-08-18 21:52:34'
tags:
- net
- c
---

The C# language's [using alias directives](https://msdn.microsoft.com/en-us/library/aa664765%28v=vs.71%29.aspx) or namespace and type aliases provide a way to disambiguate between namespaces or types with the same name. For example, both the `System.Net` and `Nancy` libraries have a type named `HttpStatusCode`. If you happen to import both namespaces in a file, then the types need to be fully qualified or else the compiler will fail with the error:

>'HttpStatusCode' is an ambiguous reference between 'Nancy.HttpStatusCode' and 'System.Net.HttpStatusCode'

Aliases help to resolve this conflict without having to fully qualify the types. In the off chance you needed both of these in the same file, you could create two aliases to avoid fully qualified names.

```
using SysNetStatusCode = System.Net.HttpStatusCode;
using NancyStatusCode = Nancy.HttpStatusCode;

NancyStatusCode statusCode1 = NancyStatusCode.OK;
SysNetStatusCode statusCode2 = SysNetStatusCode.OK;
```
###### Beyond Disambiguation
This works well and is nothing new to the average C# developer. I like to take type aliases one step further and use them to shorten long type names with more descriptive ones.

{% raw %}
```
using Coordinates = System.Tuple<decimal, decimal, decimal>;
using ConfigMap = System.Collections.Generic.Dictionary<string, SomeApp.Service.TenantConfiguration>;

Coordinates coordinates = new Coordinates(1, 2, 3);
ConfigMap configMap = new ConfigMap() {{"Tenant1", TenantConfiguration.Create()}};
```
{% endraw %}
As far as I'm aware, this only works within the scope of the current file. Even if I could use it globally, I'm not sure it would be a good idea. It would be too confusing for someone coming into the codebase for the first time (or me after a few months away). Used sparingly though, it can really clean things up.