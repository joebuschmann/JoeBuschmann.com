---
layout: post
title: 'F# Functions: Tuple Syntax'
date: '2011-07-18 19:35:42'
tags:
- net
- f
---

I'm relatively new to F# and functional programming and recently worked through an issue that had me perplexed.┬á The issue was with creating functions with the ΓÇ£tuple syntaxΓÇ¥ versus the normal syntax of separating arguments by spaces.



Consider the simple functions below that adds numeric arguments together.┬á They are constructed in two ways.┬á The first is by using a tuple to group the two arguments together.┬á It looks very similar to the syntax from other languages such as C#.┬á In the second declaration, the arguments are separated by spaces.

<pre style="font-family: consolas; background: white; color: black; font-size: 13px;"><span style="color: green;">// The arguments are grouped using a tuple.</span>

<span style="color: blue;">let</span> add1(x, y) = x + y



<span style="color: green;">// The arguments are listed separately.</span>

<span style="color: blue;">let</span> add2 x y = x + y</pre>

I initially thought that the signatures of the two functions were the same:┬á <em>int -&gt; int ΓÇô&gt; int</em>; however that isnΓÇÖt the case.┬á The signature for the first is:┬á <em>int * int ΓÇô&gt; int</em> with a tuple being passed as the single argument.┬á So the tuple syntax is more than just another way of doing the same thing.┬á A tuple is actually created, and itΓÇÖs arguments are passed to the function body.┬á I discovered this by trying to interchange the two styles when calling the methods.┬á This left me scratching my head trying to figure out where the syntax problem was.



The snippet below from the F# Interactive Window shows an attempt to call the add1 function without using the tuple syntax. Notice the error message.

<pre style="font-family: consolas; background: white; color: black; font-size: 13px;">&gt; let add1(x, y) = x + y;;



val add1 : int * int -&gt; int



&gt; add1 3 4;;

  add1 3 4;;

  ^^^^^^



stdin(2,1): error FS0003: This value is not a function and cannot be applied</pre>

Using the tuple syntax, no error is thrown, and the expected result is returned.┬á You can even create a tuple value using the let binding and pass it in.

<pre style="font-family: consolas; background: white; color: black; font-size: 13px;">&gt; add1(3, 4);; 

val it : int = 7

&gt; let args = (3, 4);;



val args : int * int = (3, 4)



&gt; add1 args;; 

val it : int = 7</pre>

Conversely, when the function is declared without the tuple syntax, its calling sites cannot use the tuple syntax.

<pre style="font-family: consolas; background: white; color: black; font-size: 13px;">&gt; let add2 x y = x + y;;



val add2 : int -&gt; int -&gt; int



&gt; add2(3, 4);;

  add2(3, 4);;

  -----^^^^



stdin(5,6): error FS0001: This expression was expected to have type

    int

but here has type

    'a * 'b

&gt; add2 3 4;; 



val it : int = 7</pre>

Because the tuple syntax resembles how you would create a method in C#, I thought it was simply another way of declaring a function with multiple arguments, but thatΓÇÖs not the case.┬á It creates a function with a single argument, a tuple, which has multiple values.



Which way is preferred?┬á Well, that depends on the situation.┬á If the two arguments are related in some way, then grouping them as a tuple makes sense.┬á If they are unrelated, then the space delimited syntax would be better so callers could take advantage of partial function application and other functional language features.