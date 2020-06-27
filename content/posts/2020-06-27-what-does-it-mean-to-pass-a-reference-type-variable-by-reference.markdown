---
layout: post
title: What Does It Mean to Pass a Reference Type Variable by Reference?
date: '2020-06-27T00:00:00Z'
tags:
- dotnet
- c-sharp
---

# Value and Reference Types

Before we get to the answer, let's review the two basic types in .NET: value types and reference types. Variables of value types directly contain their data according to the [Microsoft docs](https://docs.microsoft.com/en-us/dotnet/visual-basic/programming-guide/language-features/data-types/value-types-and-reference-types). If you assign one value type variable to another, the .NET runtime will make a copy of its data. Operations on one variable will not affect another (with one exception as we'll see). Value types are defined as structures or enumerations, and examples include the .NET numeric types, Boolean, Datetime, and Char.

Reference type variables, on the other hand, contain a reference to data stored in memory. If you assign one reference type variable to another, the runtime does not make a copy of the data that it references. Instead, the runtime copies the address of the reference which is the location of the data on the managed heap. This means two reference type variables can refer to the same information stored in memory. Reference types are defined as classes, and examples include String, Array, delegates, and most of the types in the .NET Foundation Class Library.

# Passing By Value

All method parameters are passed by value by default, even reference types. This means a copy of the data the variable contains is passed in the method parameter. For value types, the variable contains the data itself, and the method receives a private copy. Any changes to the data are not visible to the caller.

```csharp
static void Main(string[] args)
{
    int val1 = 1861;

    Console.WriteLine($"The value is {val1}.");
    
    SomeMethod(val1);
    
    Console.WriteLine($"The value is {val1}.");
}

static void SomeMethod(int val2)
{
    val2 = 1939; // Updating val2 will not affect the value of val1.
}
```

```bash
The value is 1861.
The value is 1861.
```

By default, reference types are passed by value like value types; however keep in mind the parameter passed into the method contains a copy of the memory address of the data on the managed heap. The address value remains the same, and no bytes are copied on the heap. This is an important distinction from value types. The receiving method can modify the object using a copy of the reference value, and the caller will see the change.

```csharp
static void Main(string[] args)
{
    List<int> outerList = new List<int> {1,2,3,4};

    Console.WriteLine($"The contents of the list are {string.Join(',', outerList)}.");
    
    AddFive(outerList);
    
    Console.WriteLine($"The contents of the list are {string.Join(',', outerList)}.");
}

static void AddFive(List<int> innerList)
{
    innerList.Add(5);
}
```

```bash
The contents of the list are 1,2,3,4.
The contents of the list are 1,2,3,4,5.
```

# Passing By Reference

When you pass a value type by reference, the runtime does not create a copy of the data. Instead the method parameter refers to the same data as the calling method. Changes to the value type are visible by the caller.

```csharp
static void Main(string[] args)
{
    int val1 = 1861;

    Console.WriteLine($"The value is {val1}.");
    
    SomeMethod(ref val1);
    
    Console.WriteLine($"The value is {val1}.");
}

static void SomeMethod(ref int val2)
{
    val2 = 1939; // Updating val2 will change the value of val1.
}
```

```bash
The value is 1861.
The value is 1939.
```

This brings us to the titular question: **what does it mean to pass a reference type variable by reference?**

Let's take a look at the example below. It creates a list with the initial values: 1,2,3,4. The method `ChangeToNewList` updates the argument, passed by value, to point to a new list, 5,6,7,8, but you can see in the console output that `outerList` is unaffected.

```csharp
static void Main(string[] args)
{
    List<int> outerList = new List<int> {1,2,3,4};
    
    Console.WriteLine($"The contents of the list are {string.Join(',', outerList)}.");

    ChangeToNewList(outerList);
    
    // The list remains the same despite the assignment in ChangeToNewList.
    Console.WriteLine($"The contents of the list are {string.Join(',', outerList)}.");
}

static void ChangeToNewList(List<int> innerList)
{
    // Update to a new list.
    innerList = new List<int> {5,6,7,8};
}
```

```bash
The contents of the list are 1,2,3,4.
The contents of the list are 1,2,3,4.
```

Now let's pass `outerList` by reference by adding the `ref` keyword. Changes to `outerList` will be visible by the calling method, but what is the value we're changing? It's a memory address. When `ChangeToNewList` updates the argument to point to a new memory address, the variable in the calling method is also updated to point to the new address - in other words, an entirely different object. You can see that in the example below.

```csharp
static void Main(string[] args)
{
    List<int> outerList = new List<int> {1,2,3,4};
    
    Console.WriteLine($"The contents of the list are {string.Join(',', outerList)}.");

    ChangeToNewList(ref outerList);
    
    // The list is changed.
    Console.WriteLine($"The contents of the list are {string.Join(',', outerList)}.");
}

static void ChangeToNewList(ref List<int> innerList)
{
    // Update to a new list.
    innerList = new List<int> {5,6,7,8};
}
```

```bash
The contents of the list are 1,2,3,4.
The contents of the list are 5,6,7,8.
```