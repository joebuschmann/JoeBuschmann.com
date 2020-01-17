---
layout: post
title: Singleton Access from a Container
date: '2012-12-01 23:20:49'
tags:
- c
- singleton
---

I learned a cool programming technique this week from a co-worker <a href="http://blog.kellybrownsberger.com/" target="_blank">Kelly Brownsberger</a> to enable container access to a singleton.┬á Say you have an interface IApplicationSettings exposed as a static singleton property, but you want to inject it into other classes via a container.┬á The trick is to create another class that implements IApplicationSettings and grabs a reference to the singleton in the constructor.┬á Each implementation of an interface member simply delegates the call to the corresponding member of the singleton reference.┬á Now you have a class that can be registered with the container and gets its values from the singleton.
<pre class="csharpcode">    <span class="kwrd">public</span> <span class="kwrd">interface</span> IApplicationSettings
    {
        <span class="kwrd">bool</span> AutoRefresh { get; }
        <span class="kwrd">bool</span> DisplayShoppingCart { get; }
    }

    <span class="rem">/// &lt;summary&gt;</span>
    <span class="rem">/// Singleton implementation of IApplicationSettings.</span>
    <span class="rem">/// &lt;/summary&gt;</span>
    <span class="kwrd">public</span> <span class="kwrd">class</span> ApplicationSettings : IApplicationSettings
    {
        <span class="kwrd">private</span> <span class="kwrd">static</span> IApplicationSettings _current;

        <span class="kwrd">public</span> <span class="kwrd">static</span> IApplicationSettings Current
        {
            get
            {
                <span class="kwrd">if</span> (_current == <span class="kwrd">null</span>)
                    _current = ReadSettings();

                <span class="kwrd">return</span> _current;
            }
        }

        <span class="kwrd">private</span> <span class="kwrd">static</span> IApplicationSettings ReadSettings()
        {
            <span class="rem">// Settings would actually come from a config file or DB.</span>
            <span class="kwrd">return</span> <span class="kwrd">new</span> ApplicationSettings(<span class="kwrd">true</span>, <span class="kwrd">true</span>);
        }

        <span class="kwrd">private</span> ApplicationSettings(<span class="kwrd">bool</span> autoRefresh, <span class="kwrd">bool</span> displayShoppingcart)
        {
            AutoRefresh = autoRefresh;
            DisplayShoppingCart = displayShoppingcart;
        }

        <span class="kwrd">public</span> <span class="kwrd">bool</span> AutoRefresh { get; <span class="kwrd">private</span> set; }

        <span class="kwrd">public</span> <span class="kwrd">bool</span> DisplayShoppingCart { get; <span class="kwrd">private</span> set; }
    }

    <span class="rem">/// &lt;summary&gt;</span>
    <span class="rem">/// Container-friendly or injectable instance of IApplicationSettings.</span>
    <span class="rem">/// Delegates to the singleton.</span>
    <span class="rem">/// &lt;/summary&gt;</span>
    <span class="kwrd">public</span> <span class="kwrd">class</span> ApplicationSettingsInstance : IApplicationSettings
    {
        <span class="kwrd">private</span> <span class="kwrd">readonly</span> IApplicationSettings _instance;

        <span class="kwrd">public</span> ApplicationSettingsInstance()
        {
            _instance = ApplicationSettings.Current;
        }

        <span class="kwrd">public</span> <span class="kwrd">bool</span> AutoRefresh
        {
            get { <span class="kwrd">return</span> _instance.AutoRefresh; }
        }

        <span class="kwrd">public</span> <span class="kwrd">bool</span> DisplayShoppingCart
        {
            get { <span class="kwrd">return</span> _instance.DisplayShoppingCart; }
        }
    }</pre>
This is a useful technique, but I don't like the way each implementation of an interface member in the ApplicationSettingsInstance class has to delegate to the private instance field.┬á I've always thought it would be nice if C# had a feature where you could specify that a particular interface is implemented by a private member.┬á Then the C# compiler could automatically fill in the interface implementation with calls to the private member.┬á The syntax could look something like the example below.┬á The _instance field implements the interface, and the interface members go away.
<pre class="csharpcode">    <span class="rem">/// &lt;summary&gt;</span>
    <span class="rem">/// Container-friendly or injectable instance of IApplicationSettings.</span>
    <span class="rem">/// Delegates to the singleton.</span>
    <span class="rem">/// &lt;/summary&gt;</span>
    <span class="kwrd">public</span> <span class="kwrd">class</span> ApplicationSettingsInstance : IApplicationSettings
    {
        <span class="kwrd">private</span> <span class="kwrd">readonly</span> IApplicationSettings _instance
            <span class="kwrd">handles</span> IApplicationSettings;

        <span class="kwrd">public</span> ApplicationSettingsInstance()
        {
            _instance = ApplicationSettings.Current;
        }
    }</pre>
I'm not sure if any other languages have a feature like this.┬á It seems like it would be fairly simple to implement.┬á In the meantime, I'm adding it to my fantasy feature list.