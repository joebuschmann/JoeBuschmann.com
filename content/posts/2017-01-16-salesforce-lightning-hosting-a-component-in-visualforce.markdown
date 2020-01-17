---
layout: post
title: Salesforce Lightning - Hosting a Component in Visualforce
date: '2017-01-16 08:40:00'
tags:
- salesforce
- lightning-component
- lightning
---

You may be familiar with overriding the editing experience for a Salesforce object with a custom Visualforce (VF) page. When users choose the new or edit actions for an object record, they get a customized view rather than the standard Salesforce view. I wanted to convert a custom edit view written in Visualforce over to Lightning. What I thought would take a day ended up consuming the better part of a week.

The problem is Salesforce doesn't yet allow you to override with a Lightning view. If you configure an sObject in Object Manager, there's a section labelled **Buttons, Links, and Actions**. There you can choose to override the new and edit actions with a VF page, but there is no option to select a Lightning Page or Component.

#### Hosting a Lightning Component in Visualforce

The solution that worked for me was to host the Lightning component in a VF page which could used to override the default views. Visualforce uses a feature called [Lightning Out](https://developer.salesforce.com/blogs/developer-relations/2016/02/lightning-components-visualforce-lightning.html) which acts as a bridge between a Lightning component and any web container. In this case, the container is the VF page.

The Salesforce documentation contains an excellent guide for [using Lightning Components in Visualforce pages](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/components_visualforce.htm). I recommend reading it if you haven't already. I won't go into the details except to summarize the steps below.

1. Include the Lightning JavaScript library in the Visualforce page by adding the `<apex:includeLightning/>` markup.
2. Create a Lightning App with the component dependencies.
3. Load the dependency app in the VF page.
4. Create the component and load it into a `<div>` element.

Here's what the Visualforce markup looks like for the generic sObject *customObject__c* and the component *customObjectEdit.cmp*. Note the invocation of `$Lightning.use()` to load the dependencies and `$Lightning.createComponent()` to initialize the component.

<script src="https://gist.github.com/joebuschmann/96b80a18688cd0bd1c3a3a7b4797fbde.js"></script>

Below is the markup for the component's dependency app. It exists solely to declare top-level dependencies which are read and loaded by `$Lightning.use()`.  Only one dependency `c:customObjectEdit` is explicitly listed, and its child dependencies are inferred from its markup.

<script src="https://gist.github.com/joebuschmann/b17710980c7a5bdd361b92e627a0f5f4.js"></script>

#### Passing the Record ID

In order for the page to display the correct record, the record ID needs to be passed to the Lightning component. *CustomObjectEdit.cmp* implements `force:hasRecordId` which specifies an attribute named *recordId* with a type of *Id*.

<script src="https://gist.github.com/joebuschmann/e802bb8140f4ea15c1211a0dd26167de.js"></script>

The VF page will need to supply the record ID as an attribute so the component knows what to load. Since the page declares a standard controller for `customObject__c`, the record ID can be accessed via the APEX markup `{!customObject__c.id}` and passed to the component when it is created.

<script src="https://gist.github.com/joebuschmann/83a583664c500deb456787601ffc9a80.js"></script>

#### Wiring Up the VF Host Page

One final configuration step is necessary to display the page. In the Lightning Experience setup under **Objects and Fields > Object Manager**, the Edit and New actions need to be configured to display the VF page for `customObject__c`.

Now the Lightning component will appear when editing or creating a record.

#### Issues with Force Events

There is one issue with this approach. If the Lightning component uses events like **force:showToast** or **force:navigateToObjectHome**, they will not fire in VisualForce. The code to load an application event `$A.get('e.force:navigateToObjectHome')` returns null. Force events are common in Lightning apps so this is a serious issue. Fortunately, there is a work-around which is the subject of my [next post](http://joebuschmann.com/salesforce-lightning-navigation-events-in-a-visualforce-page).