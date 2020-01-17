---
layout: post
title: The C# Language - Overflow Checking for Integral Operations
date: '2017-12-14 23:06:59'
tags:
- c
- net
---

The C# language has been around for over 15 years. It started off as a Java ripoff and evolved into its own language. Some parts of the language I use daily: enumerators, generics, async/await. Other parts lurk in the shadows until the rare moment when I need to put them to use.

One such part is overflow checking for integral operations.

#### Compiler Option
By default, integral operations are not checked for overflows either by the C# compiler or at runtime. The reason is unchecked operations are faster than their checked counterparts. You can control this behavior with the `/checked[+|-]` compiler option or with the *Check for arithmetic overflow/underflow* property in a Visual Studio project's advanced build settings.

With overflow checking off (default), adding 1 to an integer's max value will yield its min value.

<script src="https://gist.github.com/joebuschmann/3b6ceeb8223d90fe2dfb51eab06e746e.js"></script>

An exception of type `OverflowException` is thrown at runtime if the checked option is enabled.

<script src="https://gist.github.com/joebuschmann/97c7633be94602ed996ca8f90ca2b41a.js"></script>

#### Checked Blocks

These options work well for an entire assembly, but what if you need to enable overflow checking for some code blocks but leave it off for others? This is where the `checked` and `unchecked` keywords are useful. Assuming overflow checking is disabled by default, you can enable it for a section of code by adding `checked { }`.

<script src="https://gist.github.com/joebuschmann/1648ac707bb9ca96f304960931affe31.js"></script>

Similarly, you can disable checking with `unchecked { }`. Constant overflows inside an unchecked block are permitted.

<script src="https://gist.github.com/joebuschmann/855331e55689cc01ffe6421553ceb11b.js"></script>

#### Type Conversions

Overflow checking also applies to type conversions. Below a large 64-bit integer is cast to a 32-bit integer in a checked block and causes an `OverflowException` to be thrown.

<script src="https://gist.github.com/joebuschmann/c6e2112eb126a72823950ca5f908075f.js"></script>

In an unchecked block, the cast succeeds for variables as well as constants.

<script src="https://gist.github.com/joebuschmann/0e18d793d67ad43e56fccc548aa8682b.js"></script>

To wrap up, overflow checking is turned off for integers by default except for constants. You can override this behavior using the `/checked[+|-]` compiler option or the `checked` and `unchecked` keywords. Interestingly, constants can overflow inside of an unchecked block although that doesn't seem very useful.