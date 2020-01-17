---
layout: post
title: String.Empty Versus ""
date: '2015-08-06 15:22:10'
tags:
- net
- c
- best-practices-2
---

If you've been in the .NET world for any length of time, you'll eventually come across someone who claims `String.Empty` performs better than `""`. I was always skeptical of this claim because this scenario seemed like something the compiler could optimize. And because the .NET runtime manages string literals in an intern pool, my guess was they would both point to the same value. That's what I suspected anyway but didn't have any proof.

Finally, I decided to write a short program to find out the truth. In the code below, lines 10 and 11 create two empty string variables. One is initialized with `String.Empty` and the other `""`. Lines 13-17 check that both are interned, their values are equal, and the references are equal.

<script src="https://gist.github.com/joebuschmann/df3119d149505c44a60a.js"></script>

The results seem to suggest both `String.Empty` and `""` point to the same interned string literal. Just to be extra safe, I looked at the disassembly for the two assignment statements.

```
            string string1 = String.Empty;
002D047D  mov         eax,dword ptr ds:[35022A0h]  
002D0483  mov         dword ptr [ebp-40h],eax  
            string string2 = "";
002D0486  mov         eax,dword ptr ds:[35022A0h]  
002D048C  mov         dword ptr [ebp-44h],eax  
```

For each variable, the same memory address 35022A0h is moved into the register `eax` followed by another MOV instruction to save the contents of `eax` to a DWORD pointer. Like I suspected they both refer to the same memory; presumably the literal empty string in the intern pool. This shows there is no difference in the managed code produced by the compiler and thus no reason other than personal preference to choose `String.Empty` over `""`.