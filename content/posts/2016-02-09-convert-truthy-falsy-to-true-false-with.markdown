---
layout: post
title: Convert Truthy/Falsy to True/False with !!
date: '2016-02-09 13:31:37'
tags:
- javascript
---

I've been primarily coding in Javascript for a couple months now, and the language's quirks have become familiar, almost normal, to me. [IIFEs](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression), truthy/falsy, `===`, and other weird language features make sense now. When I came across `!!` in a Javascript library, I thought I'd missed some obscure syntax. Searching online provided no clues as to what it did. I read through the [list of Javascript expressions and operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators) and only found the familiar `!` logical not operator.

Finally, it hit me. `!!` is not a single operator but `!` applied twice. But what's the point? Doesn't it just return the same boolean value? The answer is yes for most programming languages, but in Javascript, it is a useful shorthand for converting truthy/falsy values into boolean true/false values. Regardless of the operand, `!` always returns a boolean value. You can use `!!` instead of a longer `if` statement or ternary operator to make the conversion. Below are some examples of this in action.

```
> !!undefined
false
> !!null
false
> !!''
false
> !!0
false
> !!'hello world'
true
> !!25
true
> !!-25
true
> !![]
true
> !!{}
true
```

Instead of `var someValue = someValue ? true : false;`, you write `var someValue = !!someValue;`. Only in Javascript!