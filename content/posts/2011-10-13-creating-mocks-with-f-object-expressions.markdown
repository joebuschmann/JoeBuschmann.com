---
layout: post
title: Creating Mocks with F# Object Expressions
date: '2011-10-13 19:59:54'
tags:
- f
- mocks
- object-expressions
- unit-testing
---

<p>When I read about F# object expressions, the first thought that popped into my head was to use them to create mocks for unit tests.&nbsp; For those of you who are not familiar with object expressions, they are similar to anonymous classes in Java.&nbsp; They are the object equivalent of lambda functions, and they allow you to create objects that implement an interface or base class without having to declare a new class.&nbsp; This is something that does not exist in C#, my primary language, where I have had to use a mock framework or define my own mock classes when writing unit tests.&nbsp; There are downsides to both solutions.&nbsp; Mock frameworks tend to be heavy-handed and are not built into the language.&nbsp; Creating custom mock classes tends to lead to a large number custom classes declared outside of the unit test methods.</p> <p>So that is why object expressions caught my eye.&nbsp; As an exercise, I decided to use them to create mocks for some common unit testing scenarios.&nbsp; First, I created a C# project with some fake business logic classes.&nbsp; Next, I added an F# project to the solution for the unit tests.&nbsp; The business logic code is listed below.&nbsp; It contains a couple of interfaces representing a service provider and state container and a class, Order, that contains the ΓÇ£logicΓÇ¥ to be unit tested.</p><pre class="csharpcode"><span class="rem">// A simple class representing a product</span>
<span class="rem">// in an order.</span>
<span class="kwrd">public</span> <span class="kwrd">class</span> Product
{
    <span class="kwrd">public</span> Product(<span class="kwrd">string</span> productName)
    {
        ProductName = productName;
    }

    <span class="kwrd">public</span> <span class="kwrd">string</span> ProductName { get; <span class="kwrd">private</span> set; }
}

<span class="rem">// A service request class.</span>
<span class="kwrd">public</span> <span class="kwrd">class</span> RetrieveAvailableProductsRequest
{
}

<span class="rem">// A service response class with a list of products.</span>
<span class="kwrd">public</span> <span class="kwrd">class</span> RetrieveAvailableProductsResponse
{
    <span class="kwrd">public</span> RetrieveAvailableProductsResponse(
        IEnumerable&lt;Product&gt; products)
    {
        Products = products;
    }

    <span class="kwrd">public</span> IEnumerable&lt;Product&gt; Products
    {
        get;
        <span class="kwrd">private</span> set;
    }
}

<span class="rem">// An interface for a service provider.</span>
<span class="kwrd">public</span> <span class="kwrd">interface</span> ICatalogService
{
    RetrieveAvailableProductsResponse
        RetrieveAvailableProducts(
            RetrieveAvailableProductsRequest request);
}

<span class="rem">// An interface for a state container.</span>
<span class="kwrd">public</span> <span class="kwrd">interface</span> IOrderState
{
    IEnumerable&lt;Product&gt; Products { get; set; }
}

<span class="rem">// A class containing simple "business logic".</span>
<span class="kwrd">public</span> <span class="kwrd">class</span> Order
{
    <span class="kwrd">public</span> IEnumerable&lt;Product&gt; GetAvailableProducts(
        ICatalogService catalogService)
    {
        var request = <span class="kwrd">new</span> RetrieveAvailableProductsRequest();
        var response =
            catalogService.RetrieveAvailableProducts(request);

        <span class="kwrd">return</span> response.Products;
    }

    <span class="kwrd">public</span> <span class="kwrd">void</span> AddProductsToOrder(
        IEnumerable&lt;Product&gt; products,
        IOrderState orderState)
    {
        <span class="kwrd">foreach</span> (var product <span class="kwrd">in</span> products)
            AddProductToState(product, orderState);
    }

    <span class="kwrd">protected</span> <span class="kwrd">virtual</span> <span class="kwrd">void</span> AddProductToState(
        Product product,
        IOrderState orderState)
    {
        var products = <span class="kwrd">new</span> List&lt;Product&gt;(orderState.Products);
        products.Add(product);

        orderState.Products = products;
    }
}</pre>
<p>There are two common mocking scenarios I come across when writing unit tests.&nbsp; The first is to use a mock object in place of one that invokes a service, database operations, file system IO, or some other long-running process.&nbsp; Both the mock and runtime objects implement an interface which is referenced by the business logic code instead of the runtime implementation.&nbsp; Unit tests can substitute or ΓÇ£injectΓÇ¥ their own implementation to avoid the expensive or unavailable runtime logic.&nbsp; This is a common pattern called dependency injection.</p>
<p>The <em>Order.AddProductsToOrder()</em> method takes an interface, <em>ICatalogService</em>, that represents a web service exposing a product catalog.&nbsp; In unit tests, the runtime implementation is replaced by a mock object that returns dummy values.&nbsp; Creating this mock object in C# requires either a mock framework or a class definition.&nbsp; Objects expressions in F# can simplify things by creating an object on the fly with very little overhead.&nbsp; Below is a simple example.</p><pre class="code"><span style="color: blue">let </span>mockProducts = seq { <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 1"</span>)
                         <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 2"</span>)
                         <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 3"</span>)
                         <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 4"</span>) }

[&lt;Fact&gt;]
<span style="color: blue">let </span>TestGetAvailableProducts() =
  <span style="color: blue">let </span>mockCatalogService = { <span style="color: blue">new </span>ICatalogService <span style="color: blue">with
    member </span>x.RetrieveAvailableProducts(request) =
      <span style="color: blue">new </span>RetrieveAvailableProductsResponse(mockProducts) }
  <span style="color: blue">let </span>order = <span style="color: blue">new </span>Order()
  <span style="color: blue">let </span>products = order.GetAvailableProducts(mockCatalogService)
  Assert.Equal(4, Seq.length(products))
  Assert.Same(mockProducts, products)</pre>
<p>The example starts by creating a sequence of dummy products.&nbsp; The mock service, <em>mockCatalogService</em>, is created on the first three lines of the unit test method using an object expression which specifies that the new object implements the <em>ICatalogService</em> interface.&nbsp; The implementation of <em>ICatalogService.RetrieveAvailableProducts()</em> returns the dummy products.&nbsp; Next, the <em>Order.GetAvailableProducts()</em> method is invoked with the mock service, and finally, the last two lines verify that the products returned by the service are the same as those returned by <em>GetAvailableProducts()</em>.</p>
<p>Below is another example where a mock is passed into Order.AddProductsToOrder() and updated.&nbsp; The function, <em>getMockOrderState()</em>, uses an object expression to create a mock implementing <em>IOrderState</em> and stores a list of products in a reference cell.</p><pre class="code"><span style="color: blue">let </span>mockProducts = seq { <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 1"</span>)
                         <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 2"</span>)
                         <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 3"</span>)
                         <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 4"</span>) }

<span style="color: blue">let </span>getMockOrderState() =
  <span style="color: blue">let </span>productsInState = ref Seq.empty
  { <span style="color: blue">new </span>IOrderState <span style="color: blue">with
    member </span>x.Products <span style="color: blue">with </span>get() = !productsInState
                      <span style="color: blue">and </span>set(value) = productsInState := value }

[&lt;Fact&gt;]
<span style="color: blue">let </span>TestAddProductsToOrderUsingMockOrderState() =
  <span style="color: blue">let </span>order = <span style="color: blue">new </span>Order()
  <span style="color: blue">let </span>mockOrderState = getMockOrderState()
  order.AddProductsToOrder(mockProducts, mockOrderState)
  Assert.Equal(4, mockOrderState.Products |&gt; Seq.length)
  mockOrderState.Products |&gt;
    Seq.iteri (<span style="color: blue">fun </span>i p <span style="color: blue">-&gt;
                 let </span>i = i + 1
                 Assert.Equal(<span style="color: maroon">"Product " </span>+ i.ToString(), p.ProductName))</pre>
<p>Sometimes it is difficult to use dependency injection because the code may not be structured correctly, and it may be too complex for easy refactoring.&nbsp; In these cases, I will often move the problematic code into its own virtual method.&nbsp; The method can be overridden in a derived class that is used in the unit tests.&nbsp; This is the second common scenario I encounter.</p><pre class="code"><span style="color: blue">let </span>mockProducts = seq { <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 1"</span>)
                         <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 2"</span>)
                         <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 3"</span>)
                         <span style="color: blue">yield new </span>Product(<span style="color: maroon">"Product 4"</span>) }

<span style="color: blue">let </span>getMockOrderState() =
  <span style="color: blue">let </span>productsInState = ref Seq.empty
  { <span style="color: blue">new </span>IOrderState <span style="color: blue">with
    member </span>x.Products <span style="color: blue">with </span>get() = !productsInState
                      <span style="color: blue">and </span>set(value) = productsInState := value }

[&lt;Fact&gt;]
<span style="color: blue">let </span>TestAddProductsToOrder() =
  <span style="color: blue">let </span>products = ref []
  <span style="color: blue">let </span>mockOrder =
    { <span style="color: blue">new </span>Order() <span style="color: blue">with
      member </span>x.AddProductToState(product, state) =
        products := product :: !products }
  <span style="color: blue">let </span>mockOrderState = getMockOrderState()
  mockOrder.AddProductsToOrder(mockProducts, mockOrderState)
  Assert.Equal(0, mockOrderState.Products |&gt; Seq.length)
  Assert.Equal(4, !products |&gt; List.length)</pre>
<p>In the example above, the <em>mockOrder</em> object overrides the <em>Order.AddProductToState()</em> method and provides a simple implementation that stores the products in a reference cell.&nbsp; This method is trivial; however, it demonstrates the idea.&nbsp; The asserts prove that the local <em>products</em> reference cell was updated and not the order state.</p>
<p>I love how object expressions make creating mocks much easier.&nbsp; This is a feature I hope makes its way into C#.&nbsp; If you want the complete source code for the examples above, you can find it <a href="https://github.com/joebuschmann/FSharpUnitTest" target="_blank">on GitHub</a>.&nbsp; You will need to install xUnit to run the unit tests.</p>