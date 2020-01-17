---
layout: post
title: The Dangers of Mutable Data
date: '2011-08-15 20:06:17'
tags:
- net
- c
---

I recently came across a bug in some C# code that would never have been a problem if the data structures being used were immutable.┬á The data consisted of a .NET dictionary with an integer key and a list as a value.┬á The details below have been changed to protect the guilty.

The offending method takes customer and product objects as parameters and retrieves a list of available pricing for the customer from a dictionary using the product ID as the key.┬á Once the list is obtained, any pricing items the customer does not qualify for are removed and the list is returned.
<pre class="code"><span style="color: blue;">private readonly </span><span style="color: #2b91af;">Dictionary</span>&lt;<span style="color: blue;">int</span>, <span style="color: #2b91af;">List</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt;&gt; _map =
    <span style="color: blue;">new </span><span style="color: #2b91af;">Dictionary</span>&lt;<span style="color: blue;">int</span>, <span style="color: #2b91af;">List</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt;&gt;();
<span style="color: blue;">private readonly </span><span style="color: #2b91af;">PricingQualificationStrategy </span>_strategy =
    <span style="color: blue;">new </span><span style="color: #2b91af;">PricingQualificationStrategy</span>();

<span style="color: blue;">private </span><span style="color: #2b91af;">List</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt; GetAvailablePricing(
    <span style="color: #2b91af;">Customer </span>customer, <span style="color: #2b91af;">Product </span>product)
{
    <span style="color: blue;">int </span>productId = product.ProductId;

    <span style="color: blue;">if </span>(_map.ContainsKey(productId))
    {
        <span style="color: #2b91af;">List</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt; potentialPricing = _map[productId];
        potentialPricing.RemoveAll(p =&gt; !_strategy.IsQualified(p, customer));
        <span style="color: blue;">return </span>potentialPricing;
    }

    <span style="color: blue;">return new </span><span style="color: #2b91af;">List</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt;();
}</pre>
The issue is with the line that removes all unqualified pricing.┬á Items are removed from the list referenced by the dictionary, not a copy, so subsequent calls to the method do not have access to the complete list.┬á If, by chance, a customer is not qualified for any pricing, then the list will be empty.┬á From that point on, the method will always return an empty list for the product regardless of customer qualification.
<pre class="code">potentialPricing.RemoveAll(p =&gt; !_strategy.IsQualified(p, customer));</pre>
It is a subtle bug.┬á When originally coded, the programmer most likely assumed the dictionary was returning a copy of the list.┬á What makes it even worse is the method returns a reference to the mutable list which can be further changed by the caller.

Instead of storing a mutable list in the dictionary, values of type <span style="color: #2b91af;">IEnumerable</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt; can be used.┬á It will achieve the desired result, returning a list of pricing items, without introducing the possibility of the list changing unexpectedly.┬á In addition, the <span style="color: #2b91af;">Enumerable</span>.Where() extension method in the System.Linq namespace can be used to return a read-only enumeration with only the qualified items.┬á The returned enumeration is a copy, and the original is unaffected.

The updated code is below.
<pre class="code"><span style="color: blue;">private readonly </span><span style="color: #2b91af;">Dictionary</span>&lt;<span style="color: blue;">int</span>, <span style="color: #2b91af;">IEnumerable</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt;&gt; _map =
    <span style="color: blue;">new </span><span style="color: #2b91af;">Dictionary</span>&lt;<span style="color: blue;">int</span>, <span style="color: #2b91af;">IEnumerable</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt;&gt;();
<span style="color: blue;">private readonly </span><span style="color: #2b91af;">PricingQualificationStrategy </span>_strategy =
    <span style="color: blue;">new </span><span style="color: #2b91af;">PricingQualificationStrategy</span>();

<span style="color: blue;">private </span><span style="color: #2b91af;">IEnumerable</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt; GetAvailablePricing(
    <span style="color: #2b91af;">Customer </span>customer, <span style="color: #2b91af;">Product </span>product)
{
    <span style="color: blue;">int </span>productId = product.ProductId;

    <span style="color: blue;">if </span>(_map.ContainsKey(productId))
    {
        <span style="color: blue;">var </span>potentialPricing = _map[productId];
        <span style="color: blue;">return </span>potentialPricing.Where(p =&gt; !_strategy.IsQualified(p, customer));
    }

    <span style="color: blue;">return new </span><span style="color: #2b91af;">List</span>&lt;<span style="color: #2b91af;">IPricing</span>&gt;();
}</pre>
This is a simple yet instructive example of the hidden dangers of mutable data.