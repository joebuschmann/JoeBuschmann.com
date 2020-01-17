---
layout: post
title: Callbacks As Lightning Component Attributes
date: '2016-10-25 14:15:44'
tags:
- salesforce
- lightning-component
- lightning
---

Last week I posed a question on [salesforce.stackexchange.com](http://salesforce.stackexchange.com/questions/144087/how-do-i-pass-a-function-as-a-lightning-component-attribute) asking how to pass a function as an attribute in a Lightning Component. As a newbie to the Lightning Component framework (and the Salesforce platform), I found it odd callbacks were not included as one of the [supported attribute types](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/ref_aura_attribute.htm). The dearth of responses to my question surprised me. Eventually I found the answer when researching [Component Events](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/events_component.htm).

#### Lightning Events

Before I elaborate, I have to say the Lightning framework's approach to callbacks or events seems excessive. My previous two web apps used React which has no separate concept of an event. React treats callback functions as any other data, and they are passed through the component hierarchy as attributes. The Lightning approach is to separate callbacks and data (strings, integers, arrays, object). Callbacks are encapsulated into events, and data are defined through strongly typed attributes.

I would prefer the event concept go away altogether and be replaced with a simple callback or function attribute type. Unfortunately, that's not how Lightning works, so I won't belabor the point any further. Instead I'll focus on how you can pass callbacks to your Lightning components using events.

#### myButton.cmp

To help illustrate my thoughts, below is a Lightning component called **myButton**. It's a wrapper around the standard `<button>` element that adds behavior to each click event. The behavior could be tracking for marketing purposes or any other common function. **myButton** exposes a single event named *press*.

**pressEvent.evt**

```
<aura:event type="COMPONENT" />
```

**myButton.cmp**

```
<aura:component>
    <aura:attribute name="label" required="true" type="String" />
    <aura:attribute name="class" required="false" type="String" />

    <!-- Declare the press event of type "pressEvent" -->
    <aura:registerEvent name="press" type="c:pressEvent"/>

    <input type="button" value="{!v.label}" class="{!v.class}" onclick="{!c.onClick}" />
</aura:component>
```

**myButtonController.js**

```
({
    onClick : function(component, event, helper) {
    	// Do some cross-cutting function like track clicks for analytics
        helper.trackButtonClick(component, event);

        var event = component.getEvent('press');
        event.fire();
    }
})
```

#### &lt;aura:handler&gt;

One way to handle the *press* event is with an instance of the `<aura:handler>` element.

**myButtonConsumer.cmp**

```
<aura:component>
	<aura:handler name="press" event="c:pressEvent" action="{!c.onPress}"/>

    <c:myButton aura:id="okButton" label="OK" class="slds-button" />
    <c:myButton aura:id="cancelButton" label="Cancel" class="slds-button" />
</aura:component>
```

**myButtonConsumerController.js**

```
({
    onPress: function(component, event) {
        alert('Button pressed!');
    }
})
```

In **myButtonConsumer.cmp**, the handler is declared with a name, event, and action. It wires up the onPress method in the controller to the OK and Cancel buttons' press events.

One disadvantage of `<aura:handler>` is there is no easy way to segment the handler function by the source component. In this example, the same handler action is invoked for both buttons. You have to write additional code to determine which component triggered the event.

```
({
    onPress: function(component, event) {
        if (event.getSource() === component.find('okButton')) {
        	alert ('OK pressed!');
        } else if (event.getSource() === component.find('cancelButton')) {
        	alert('Cancel pressed!');
        }
    }
})
```

Of course, there are times when you would want all press events routed through the same handler, but that would be the exception rather than the rule.

#### Callbacks As Attributes

The alternative to `<aura:handler>` is to pass a callback as an attribute. Unfortunately, this isn't clear from the official documentation. I discovered it in the answer to [an unrelated Stack Exchange question](http://salesforce.stackexchange.com/questions/89757/aurahandler-has-invalid-name-attribute-value#89759). Essentially, you can pass event handlers to `<aura:registerEvent>` the same way you pass data to `<aura:attribute>`. I'm not sure why the Salesforce documentation doesn't emphasize this more.

With this in mind, you can rewrite **myButtonConsumer** as follows:

**myButtonConsumer.cmp**

```
<aura:component>
    <!-- No aura:handler necessary. Just pass the callback as an attribute -->
    <!--<aura:handler name="press" event="c:pressEvent" action="{!c.onOk}"/>-->

    <c:myButton aura:id="okButton" label="OK" class="slds-button" press="{!c.onOk}" />
    <c:myButton aura:id="cancelButton" label="Cancel" class="slds-button" press="{!c.onCancel}" />
</aura:component>
```

**myButtonConsumerController.js**

```
({
    onOk: function(component, event) {
        alert('OK pressed!');
    },

    onCancel: function(component, event) {
        alert('Cancel pressed!');
    }
})
```

That's it. The `press` event is wired up to the controller via an attribute. The `<aura:handler>` element goes away. I much prefer this approach. It requires one fewer element and better encapsulates component markup.

#### Where's the Documentation?

I'm not sure why Saleforce chooses not to document the second approach to event handling. Maybe it is deprecated? I doubt it. My best guess is `<aura:handler>` is their preferred way.

If anyone has an answer, I'd love to hear from you in the comments.
