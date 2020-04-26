---
layout: post
title: Running SpecFlow Scenarios in Parallel
date: '2018-08-24 22:48:34'
tags:
- specflow
- c-sharp
- dotnet
---

As of SpecFlow version 2.0, you can run scenarios in parallel. This means faster execution times and faster feedback in your continuous integration process.

### Memory Isolation

To enable parallel execution, you must use a test runner that supports it. Available runners include NUnit 3.0, xUnit 2.0, and the SpecFlow+ Runner (specrun). Specrun is a commercial product, but it has advanced features like memory isolation via an app domain or process. NUnit and xUnit don't support memory isolation, so they requre your tests to be thread safe. Also, you won't be able to use the static context properties `ScenarioContext.Current`, `FeatureContext.Current`, and `ScenarioStepContext.Current`. The following code throws a `SpecFlowException` when run in parallel.

```csharp
private readonly ScenarioContext _scenarioContext = ScenarioContext.Current;
```

You can work around this limitation by using dependency injection. In fact, you should use DI anyway for a cleaner scalable code base. See my post on [Reusable Bindings in SpecFlow](/posts/2018/08/reusable-bindings-in-specflow) for more details on leveraging SpecFlow's IoC container.

```csharp
private readonly ScenarioContext _scenarioContext;

public AddressSteps(ScenarioContext scenarioContext)
{
    _scenarioContext = scenarioContext;
}
```

### NUnit 3 Support

NUnit 3 requires the assembly-level attribute `Parallelizable` to configure [parallel test execution](https://github.com/nunit/docs/wiki/Parallelizable-Attribute).

```csharp
using NUnit.Framework;
[assembly: Parallelizable(ParallelScope.Fixtures)]
```

### Hooks

Hooks or event bindings behave the same except for one crucial difference: `BeforeFeature` and `AfterFeature` hooks will execute multiple times if scenarios from the same feature run in parallel. They should be thread-safe and safe to execute repeatedly. From the documentation:

> Each thread manages its own enter/exit feature execution workflow. The `[BeforeFeature]` and `[AfterFeature]` hooks may be executed multiple times in different threads if the different threads run scenarios from the same feature file. The execution of these hooks do not block one another, but the Before/After feature hooks are called in pairs within a single thread (the `[BeforeFeature]` hook of the next scenario is only executed after the `[AfterFeature]` hook of the previous one). Each thread has a separate (and isolated) FeatureContext.
> \- [SpecFlow Documentation](https://specflow.org/documentation/Parallel-Execution/)

<hr />

Enabling parallel execution in SpecFlow is pretty straightforward. If you're converting an existing test suite, you should set aside time to work through failures due to race conditions and lack of thread-safety. It could take a few weeks for a large number of scenarios.