---
layout: post
title: The C# Language - Overflow Checking for Integral Operations
slug: the-csharp-language-overflow-checking-for-integral-operations
date: '2017-12-14 23:06:59'
tags:
- c-sharp
- dotnet
---

![Waterfall](/images/waterfall.jpg#c)

The C# language has been around for over 15 years. It started off as a Java ripoff and evolved into its own language. Some parts of the language I use daily: enumerators, generics, async/await. Other parts lurk in the shadows until the rare moment when I need to put them to use.

One such part is overflow checking for integral operations.

# Compiler Option
By default, integral operations are not checked for overflows either by the C# compiler or at runtime. The reason is unchecked operations are faster than their checked counterparts. You can control this behavior with the `/checked[+|-]` compiler option or with the *Check for arithmetic overflow/underflow* property in a Visual Studio project's advanced build settings.

With overflow checking off (default), adding 1 to an integer's max value will yield its min value.

```csharp
[Test]
public void ValidateOverflowWithCompilerOptionUnchecked()
{
    // Compile time error regardless of compiler option.
    //const int BigConstant = int.MaxValue + 1;

    int i = int.MaxValue;
    i++;
    Assert.AreEqual(int.MinValue, i, "No exception should be thrown when overflow checking is off.");

    uint j = uint.MaxValue;
    j++;
    Assert.AreEqual(uint.MinValue, j, "No exception should be thrown when overflow checking is off.");
}
```

An exception of type `OverflowException` is thrown at runtime if the checked option is enabled.

```csharp
[Test]
public void ValidateOverflowWithCompilerOptionChecked()
{
    // Compile time error regardless of compiler option.
    //const int BigConstant = int.MaxValue + 1;

    int i = int.MaxValue;
    Assert.Throws<OverflowException>(() => i++,
        "An exception should be thrown when overflow checking is on.");

    uint j = uint.MaxValue;
    Assert.Throws<OverflowException>(() => j++,
        "An exception should be thrown when overflow checking is on.");
}
```

# Checked Blocks

These options work well for an entire assembly, but what if you need to enable overflow checking for some code blocks but leave it off for others? This is where the `checked` and `unchecked` keywords are useful. Assuming overflow checking is disabled by default, you can enable it for a section of code by adding `checked { }`.

```csharp
[Test]
public void ValidateOverflowWithCheckedBlock()
{
    int i = int.MaxValue;
    i++;
    Assert.AreEqual(int.MinValue, i, "Overflow checking is off by default.");

    checked
    {
        i = int.MaxValue;
        Assert.Throws<OverflowException>(() => i++,
            "An exception should be thrown when overflow checking is on.");
    }
}
```

Similarly, you can disable checking with `unchecked { }`. Constant overflows inside an unchecked block are permitted.

```csharp
[Test]
public void ValidateOverflowWithUncheckedBlock()
{
    checked
    {
        int i = int.MaxValue;
        Assert.Throws<OverflowException>(() => i++,
            "An exception should be thrown when overflow checking is on.");

        unchecked
        {
            i = int.MaxValue;
            i++;
            Assert.AreEqual(int.MinValue, i, "Overflow checking is off.");

            // What about constants?
            const int BigConstant = int.MaxValue + 1;
            Assert.AreEqual(int.MinValue, BigConstant,
                "Overflow checking is ignored for constants in an unchecked block.");
        }
    }
}
```

# Type Conversions

Overflow checking also applies to type conversions. Below a large 64-bit integer is cast to a 32-bit integer in a checked block and causes an `OverflowException` to be thrown.

```csharp
[Test]
public void ValidateTypeConversionOverflowWithCheckedBlock()
{
    long l = int.MaxValue;
    l++;

    int i = (int)l;
    Assert.AreEqual(int.MinValue, i, "Overflow checking is off by default.");

    checked
    {
        Assert.Throws<OverflowException>(() => i = (int) l,
            "An exception should be thrown when overflow checking is on.");
    }
}
```

In an unchecked block, the cast succeeds for variables as well as constants.

```csharp
[Test]
public void ValidateTypeConversionOverflowWithUncheckedBlock()
{
    checked
    {
        long l = int.MaxValue;
        l++;

        int i;
        Assert.Throws<OverflowException>(() => i = (int)l,
            "An exception should be thrown when overflow checking is on.");

        unchecked
        {
            i = (int) l;
            Assert.AreEqual(int.MinValue, i, "Overflow checking is off.");

            // What if we increment l and try again?
            l++;
            i = (int)l;
            Assert.AreEqual(int.MinValue + 1, i, "The value continues to overflow in the negative range.");

            // What about constants?
            const int BigConstant = (int)long.MaxValue;
            Assert.AreEqual(-1, BigConstant,
                "Overflow checking is ignored for constants in an unchecked block.");
        }
    }
}
```

To wrap up, overflow checking is turned off for integers by default except for constants. You can override this behavior using the `/checked[+|-]` compiler option or the `checked` and `unchecked` keywords. Interestingly, constants can overflow inside of an unchecked block although that doesn't seem very useful.