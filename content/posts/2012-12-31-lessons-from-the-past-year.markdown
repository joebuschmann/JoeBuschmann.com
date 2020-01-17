---
layout: post
title: Lessons from the Past Year
date: '2012-12-31 14:54:43'
tags:
- c
- best-practices-2
---

As 2012 comes to a close, it's time to look back on some of the things I've learned. During the past year, my focus has been on enterprise service development, and thanks to a talented software architect and co-worker <a href="http://blog.kellybrownsberger.com/" target="_blank">Kelly Brownsberger</a>, I feel much more confident with my development skills. Together we focused on re-architecting and refactoring large portions of an ordering service. Now, looking back, three lessons in particular have stuck with me and changed the way I develop software.

They are 1) encapsulate operations vertically not horizontally, 2) everything's a service, and 3) separate contract classes from business logic classes. In the remainder of this blog post I expand on these ideas and try to explain why they have made such an impression.
<h3>Encapsulate Operations Vertically Not Horizontally</h3>
For web service implementations, encapsulation at the operation level (vertically) rather than at the service level (horizontally) seems to work better over the long run. Having an implementation class per operation makes refactoring, unit testing, and reuse much easier.

For example, let's say a hypothetical product ordering service implemented with WCF has a contract IOrderService with several operations and an implementing class OrderService. Most developers would immediately start implementing the service in the OrderService class. Over time OrderService would continue to accumulate implementing code with common code among the operations separated out into private methods or separate classes. Eventually it evolves into a bulky god class. The operations become hard to unit test because it's difficult to know which methods, fields, or private members a particular operation needs. I've seen this happen several times where I work.
<pre class="csharpcode">[ServiceContract]
<span class="kwrd">public</span> <span class="kwrd">interface</span> IOrderService
{
    [OperationContract]
    InitiateOrderResponse InitiateOrder(
        InitiateOrderRequest request);

    [OperationContract]
    CommitOrderResponse CommitOrder(
        CommitOrderRequest request);

    [OperationContract]
    AddProductToOrderResponse AddProductToOrder(
        AddProductToOrderRequest request);

    [OperationContract]
    RemoveProductFromOrderResponse RemoveProductFromOrder(
        RemoveProductFromOrderRequest request);
}

[ServiceBehavior]
<span class="kwrd">public</span> <span class="kwrd">class</span> OrderService : IOrderService
{
    <span class="kwrd">public</span> InitiateOrderResponse InitiateOrder(
        InitiateOrderRequest request)
    {
        <span class="rem">// Lots of implementing code here.</span>
    }

    <span class="kwrd">public</span> CommitOrderResponse CommitOrder(
        CommitOrderRequest request)
    {
        <span class="rem">// Lots of implementing code here.</span>
    }

    <span class="kwrd">public</span> AddProductToOrderResponse AddProductToOrder(
        AddProductToOrderRequest request)
    {
        <span class="rem">// Lots of implementing code here.</span>
    }

    <span class="kwrd">public</span> RemoveProductFromOrderResponse RemoveProductFromOrder(
        RemoveProductFromOrderRequest request)
    {
        <span class="rem">// Lots of implementing code here.</span>
    }
}</pre>
As an alternative, this service could be implemented with a class per operation. Each method simply creates an instance of the operation (or retrieves it from a container like Windsor) and executes it. Any common code between the operations can be pulled out into a separate class. For example, they all may return a shopping cart with the latest view of the order. The cart generation code can be encapsulated into the GetShoppingCartTask class and shared among the operations.

This setup is much easier to follow and unit test. Each operation contains just the code it needs to do its job (<a href="http://en.wikipedia.org/wiki/Single_responsibility_principle" target="_blank">Single Responsibility Principle</a>).

Also, the operations are also easier to compose. The InitiateOrder operation may take one or more initial products and can simply invoke the AddProductToOrderOperation class to add them to the order.
<pre class="csharpcode">[ServiceContract]
<span class="kwrd">public</span> <span class="kwrd">interface</span> IOrderService
{
    [OperationContract]
    InitiateOrderResponse InitiateOrder(
        InitiateOrderRequest request);

    [OperationContract]
    CommitOrderResponse CommitOrder(
        CommitOrderRequest request);

    [OperationContract]
    AddProductToOrderResponse AddProductToOrder(
        AddProductToOrderRequest request);

    [OperationContract]
    RemoveProductFromOrderResponse RemoveProductFromOrder(
        RemoveProductFromOrderRequest request);
}

[ServiceBehavior]
<span class="kwrd">public</span> <span class="kwrd">class</span> OrderService : IOrderService
{
    <span class="kwrd">public</span> InitiateOrderResponse InitiateOrder(
        InitiateOrderRequest request)
    {
        var operation = <span class="kwrd">new</span> InitiateOrderOperation();
        <span class="kwrd">return</span> operation.Execute(request);
    }

    <span class="kwrd">public</span> CommitOrderResponse CommitOrder(
        CommitOrderRequest request)
    {
        var operation = <span class="kwrd">new</span> CommitOrderOperation();
        <span class="kwrd">return</span> operation.Execute(request);
    }

    <span class="kwrd">public</span> AddProductToOrderResponse AddProductToOrder(
        AddProductToOrderRequest request)
    {
        var operation = <span class="kwrd">new</span> AddProductToOrderOperation();
        <span class="kwrd">return</span> operation.Execute(request);
    }

    <span class="kwrd">public</span> RemoveProductFromOrderResponse RemoveProductFromOrder(
        RemoveProductFromOrderRequest request)
    {
        var operation = <span class="kwrd">new</span> RemoveProductFromOrderOperation();
        <span class="kwrd">return</span> operation.Execute(request);
    }
}

<span class="kwrd">public</span> <span class="kwrd">class</span> RemoveProductFromOrderOperation
{
    <span class="kwrd">public</span> RemoveProductFromOrderResponse Execute(
        RemoveProductFromOrderRequest request)
    {
        <span class="rem">// Implementation here.</span>
    }
}

<span class="kwrd">public</span> <span class="kwrd">class</span> AddProductToOrderOperation
{
    <span class="kwrd">public</span> AddProductToOrderResponse Execute(
        AddProductToOrderRequest request)
    {
        <span class="rem">// Implementation here.</span>
    }
}

<span class="kwrd">public</span> <span class="kwrd">class</span> CommitOrderOperation
{
    <span class="kwrd">public</span> CommitOrderResponse Execute(
        CommitOrderRequest request)
    {
        <span class="rem">// Implementation here.</span>
    }
}

<span class="kwrd">public</span> <span class="kwrd">class</span> InitiateOrderOperation
{
    <span class="kwrd">public</span> InitiateOrderResponse Execute(
        InitiateOrderRequest request)
    {
        <span class="rem">// Implementation here.</span>
    }
}</pre>
<h3>Everything's a Service</h3>
Most web services I've seen implement a standard <a href="http://en.wikipedia.org/wiki/Data_transfer_object" target="_blank">Data Transfer Object (DTO)</a> pattern for their operations utilizing request and response DTOs or messages. The request message contains all of the information necessary for the operation to do its job, and the response message contains all of the operation's output. The behavior of a message class is limited to retrieving and saving its own data.

DTOs make unit testing much easier by increasing the visibility into what the operation does. Everything it needs to execute comes in the request, and the data the test needs to verify is returned in the response.

This works well for operations, so I thought why not use this pattern for business objects as well? Essentially each class can be treated like a service with its own well-defined contract and messages. Taking this idea even further, each public method could become stateless as private data members are moved to the request/response classes.

About two years ago I started converting business objects over to this pattern in one of my company's large web services. I even updated private methods to remove dependencies on class fields. Instead each method received these values as arguments. This year I continued this process, and it has worked very well. The guts of the service are now more unit testable and composable.
<h3>Separate Contract Classes from Business Logic Classes</h3>
I like to create a strict separation between service contract classes and implementation classes, and I'm accustomed to putting contract classes into their own assembly. Occasionally, I'll notice an implementation specific detail in the contract.┬á Sometimes it's a simple field to hold some temporary data.┬á Other times it's one or more methods with internal state. This drives me crazy because it becomes difficult to determine which data consuming clients are dependent on.

Below is an example of what I mean. It is based on something I came across recently. ShoppingCartItem is a contract class for a WCF service. It exposes Name and Price as data members. Two additional properties IsDiscountIncludedInPrice and IsPenaltyIncludedInPrice are ignored by the data contract serializer as instructed by the XmlIgnore attribute.┬á The presence of this attribute in a contract class is a clear code smell. To make matters worse, a couple of methods have been added.
<pre class="csharpcode">[DataContract]
<span class="kwrd">public</span> <span class="kwrd">class</span> ShoppingCartItem
{
    [DataMember]
    <span class="kwrd">public</span> <span class="kwrd">string</span> Name { get; set; }

    [DataMember]
    <span class="kwrd">public</span> <span class="kwrd">string</span> Price { get; set; }

    [XmlIgnore]
    <span class="kwrd">public</span> <span class="kwrd">bool</span> IsDiscountIncludedInPrice { get; set; }

    [XmlIgnore]
    <span class="kwrd">public</span> <span class="kwrd">bool</span> IsPenaltyIncludedInPrice { get; set; }

    <span class="kwrd">public</span> <span class="kwrd">void</span> AddDiscount(Disount discount)
    {
        <span class="rem">// Logic to update the price with a discount.</span>
    }

    <span class="kwrd">public</span> <span class="kwrd">void</span> AddPenalty(Penalty penalty)
    {
        <span class="rem">// Logic to update the price with a monetary penalty.</span>
    }
}</pre>
To fix this, I did three things. First, I created a class called ShoppingCartItemImp (a terrible name I know) with all of ShoppingCartItem's members. Next, I removed the members in ShoppingCartItem that are not a part of the contract. Finally, I added a method to ShoppingCartItemImp that returns an instance of ShoppingCartItem.

The result is a new class whose implementation can change as the service logic evolves and a contract class free of implementation details.
<pre class="csharpcode">[DataContract]
<span class="kwrd">public</span> <span class="kwrd">class</span> ShoppingCartItem
{
    [DataMember]
    <span class="kwrd">public</span> <span class="kwrd">string</span> Name { get; set; }

    [DataMember]
    <span class="kwrd">public</span> <span class="kwrd">string</span> Price { get; set; }
}

<span class="kwrd">public</span> <span class="kwrd">class</span> ShoppingCartItemImp
{
    <span class="kwrd">public</span> <span class="kwrd">string</span> Name { get; set; }

    <span class="kwrd">public</span> <span class="kwrd">string</span> Price { get; set; }

    <span class="kwrd">public</span> <span class="kwrd">bool</span> IsDiscountIncludedInPrice { get; set; }

    <span class="kwrd">public</span> <span class="kwrd">bool</span> IsPenaltyIncludedInPrice { get; set; }

    <span class="kwrd">public</span> ShoppingCartItem ToShoppingCartItem()
    {
        <span class="kwrd">return</span> <span class="kwrd">new</span> ShoppingCartItem()
            {
                Name = <span class="kwrd">this</span>.Name,
                Price = <span class="kwrd">this</span>.Price
            };
    }

    <span class="kwrd">public</span> <span class="kwrd">void</span> AddDiscount(
        ShoppingCartItemImp shoppingCartItemImp,
        Disount discount)
    {
        <span class="rem">// Logic to update the price with a discount.</span>
    }

    <span class="kwrd">public</span> <span class="kwrd">void</span> AddPenalty(
        ShoppingCartItemImp shoppingCartItemImp,
        Penalty penalty)
    {
        <span class="rem">// Logic to update the price with a monetary penalty.</span>
    }
}</pre>
<h3>Onward to 2013...</h3>
These are not the only lessons I've learned but the ones that have made the biggest impression. Others include decoupling logic with an event bus, finally getting comfortable with containers, and favoring automated tests over a UI for testing.

Here's hoping to an equally productive 2013.