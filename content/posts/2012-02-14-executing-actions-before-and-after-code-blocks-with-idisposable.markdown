---
layout: post
title: Executing Actions Before and After Code Blocks with IDisposable
date: '2012-02-14 21:06:32'
tags:
- c
- best-practices-2
- f
---

<p>I ran into a scenario this week where a boolean field was being flipped temporarily to modify behavior elsewhere while a block of code was executing.&nbsp; I have seen this pattern mainly in WinForms applications where data are being loaded into controls, but their events need to be suppressed during the load.&nbsp; Afterward, the events should fire normally.</p> <p>Below is an example of this scenario.&nbsp; It is a Windows form with a single combobox.&nbsp; Whenever the selected value of the combobox changes, an expensive action is executed such as a database or service call.&nbsp; During the loading of the form data, the possible values of the combobox are added, and the first item is selected.&nbsp; This causes the expensive operation to execute unnecessarily.</p><pre class="csharpcode">    <span class="kwrd">public</span> <span class="kwrd">partial</span> <span class="kwrd">class</span> BeforeAndAfterTest : Form
    {
        <span class="kwrd">public</span> BeforeAndAfterTest()
        {
            InitializeComponent();
        }

        <span class="kwrd">protected</span> <span class="kwrd">override</span> <span class="kwrd">void</span> OnLoad(EventArgs e)
        {
            LoadComboBoxData();

            <span class="kwrd">base</span>.OnLoad(e);
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> LoadComboBoxData()
        {
            comboBox.Items.Add(<span class="str">"Item 1"</span>);
            comboBox.Items.Add(<span class="str">"Item 2"</span>);
            comboBox.Items.Add(<span class="str">"Item 3"</span>);
            comboBox.Items.Add(<span class="str">"Item 4"</span>);

            comboBox.SelectedIndex = 0;
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> comboBox_SelectedIndexChanged(
            <span class="kwrd">object</span> sender, EventArgs e)
        {
            PerformExpensiveOperation();
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> PerformExpensiveOperation()
        {
            <span class="rem">// Imagine that this method performs an expensive operation</span>
            <span class="rem">// that doesn't need to execute when the form loads.</span>
            MessageBox.Show(<span class="str">"Expensive operation."</span>);
        }
    }</pre>
<p>A common fix for this issue is to add a boolean flag to indicate that the event action does not need to execute.</p><pre class="csharpcode">    <span class="kwrd">public</span> <span class="kwrd">partial</span> <span class="kwrd">class</span> BeforeAndAfterTest : Form
    {
        <span class="highlight"><span class="kwrd">private</span> <span class="kwrd">bool</span> _loading = <span class="kwrd">false</span>;</span>

        <span class="kwrd">public</span> BeforeAndAfterTest()
        {
            InitializeComponent();
        }

        <span class="kwrd">protected</span> <span class="kwrd">override</span> <span class="kwrd">void</span> OnLoad(EventArgs e)
        {
            <span class="highlight">_loading = <span class="kwrd">true</span>;</span>

            LoadComboBoxData();

            <span class="kwrd">base</span>.OnLoad(e);

            <span class="highlight">_loading = <span class="kwrd">false</span>;</span>
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> LoadComboBoxData()
        {
            comboBox.Items.Add(<span class="str">"Item 1"</span>);
            comboBox.Items.Add(<span class="str">"Item 2"</span>);
            comboBox.Items.Add(<span class="str">"Item 3"</span>);
            comboBox.Items.Add(<span class="str">"Item 4"</span>);

            comboBox.SelectedIndex = 0;
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> comboBox_SelectedIndexChanged(
            <span class="kwrd">object</span> sender, EventArgs e)
        {
            <span class="highlight"><span class="kwrd">if</span> (_loading)</span>
                <span class="highlight"><span class="kwrd">return</span>;</span>

            PerformExpensiveOperation();
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> PerformExpensiveOperation()
        {
            <span class="rem">// Imagine that this method performs an expensive operation</span>
            <span class="rem">// that doesn't need to execute when the form loads.</span>
            MessageBox.Show(<span class="str">"Expensive operation."</span>);
        }
    }</pre>
<p>This type of coding makes me nervous because a programmer could forget to wrap the updates to the boolean flag in a <em>try {} finally {}</em> block as in the previous example.&nbsp; In the case that an exception is thrown, the flag will not be reset resulting in incorrect behavior.</p>
<p>A nice solution is to use a class that implements <em>IDisposable</em> to update the flag before and after the block of code is run.</p><pre class="csharpcode">    <span class="kwrd">public</span> <span class="kwrd">class</span> BeforeAndAfterBlock : IDisposable
    {
        <span class="kwrd">private</span> <span class="kwrd">readonly</span> Action _before;
        <span class="kwrd">private</span> <span class="kwrd">readonly</span> Action _after;

        <span class="kwrd">public</span> BeforeAndAfterBlock(Action before, Action after)
        {
            _before = before ?? <span class="kwrd">delegate</span> { };
            _after = after ?? <span class="kwrd">delegate</span> { };

            _before();
        }

        <span class="kwrd">public</span> <span class="kwrd">void</span> Dispose()
        {
            _after();
        }
    }</pre>
<p>The constructor takes two actions to execute.&nbsp; One is executed immediately, and the other is executed in the <em>Dispose()</em> method.&nbsp; With the C# using statement, the code becomes clear, concise, and most importantly, safer.</p><pre class="csharpcode">    <span class="kwrd">public</span> <span class="kwrd">partial</span> <span class="kwrd">class</span> BeforeAndAfterTest : Form
    {
        <span class="kwrd">private</span> <span class="kwrd">bool</span> _loading = <span class="kwrd">false</span>;

        <span class="kwrd">public</span> BeforeAndAfterTest()
        {
            InitializeComponent();
        }

        <span class="kwrd">protected</span> <span class="kwrd">override</span> <span class="kwrd">void</span> OnLoad(EventArgs e)
        {
            <div class="highlight">            <span class="kwrd">using</span> (<span class="kwrd">new</span> BeforeAndAfterBlock(
                () =&gt; _loading = <span class="kwrd">true</span>, () =&gt; _loading = <span class="kwrd">false</span>))</div>
            {
                LoadComboBoxData();

                <span class="kwrd">base</span>.OnLoad(e);
            }
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> LoadComboBoxData()
        {
            comboBox.Items.Add(<span class="str">"Item 1"</span>);
            comboBox.Items.Add(<span class="str">"Item 2"</span>);
            comboBox.Items.Add(<span class="str">"Item 3"</span>);
            comboBox.Items.Add(<span class="str">"Item 4"</span>);

            comboBox.SelectedIndex = 0;
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> comboBox_SelectedIndexChanged(
            <span class="kwrd">object</span> sender, EventArgs e)
        {
            <span class="kwrd">if</span> (_loading)
                <span class="kwrd">return</span>;

            PerformExpensiveOperation();
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> PerformExpensiveOperation()
        {
            <span class="rem">// Imagine that this method performs an expensive operation</span>
            <span class="rem">// that doesn't need to execute when the form loads.</span>
            MessageBox.Show(<span class="str">"Expensive operation."</span>);
        }
    }</pre>
<p>For further refinement, <em>BeforeAndAfterBlock</em> could be subclassed to simplify its construction and make it easier to use in multiple locations.&nbsp; Also, instead of using a flag, the event handler could be detached from the event for the duration of the block and then re-attached.</p><pre class="csharpcode">    <span class="kwrd">public</span> <span class="kwrd">partial</span> <span class="kwrd">class</span> BeforeAndAfterTest : Form
    {
        <span class="kwrd">public</span> BeforeAndAfterTest()
        {
            InitializeComponent();
        }

        <span class="kwrd">protected</span> <span class="kwrd">override</span> <span class="kwrd">void</span> OnLoad(EventArgs e)
        {
            <span class="highlight"><span class="kwrd">using</span> (<span class="kwrd">new</span> LoadingBlock(<span class="kwrd">this</span>))</span>
            {
                LoadComboBoxData();

                <span class="kwrd">base</span>.OnLoad(e);
            }
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> LoadComboBoxData()
        {
            comboBox.Items.Add(<span class="str">"Item 1"</span>);
            comboBox.Items.Add(<span class="str">"Item 2"</span>);
            comboBox.Items.Add(<span class="str">"Item 3"</span>);
            comboBox.Items.Add(<span class="str">"Item 4"</span>);

            comboBox.SelectedIndex = 0;
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> comboBox_SelectedIndexChanged(
            <span class="kwrd">object</span> sender, EventArgs e)
        {
            PerformExpensiveOperation();
        }

        <span class="kwrd">private</span> <span class="kwrd">void</span> PerformExpensiveOperation()
        {
            <span class="rem">// Imagine that this method performs an expensive operation</span>
            <span class="rem">// that doesn't need to execute when the form loads.</span>
            MessageBox.Show(<span class="str">"Expensive operation."</span>);
        }

        <div class="highlight">        <span class="kwrd">private</span> <span class="kwrd">class</span> LoadingBlock : BeforeAndAfterBlock
        {
            <span class="kwrd">public</span> LoadingBlock(BeforeAndAfterTest form)
                : <span class="kwrd">base</span>(() =&gt; DetachEvents(form), () =&gt; AttachEvents(form))
            {
            }

            <span class="kwrd">private</span> <span class="kwrd">static</span> <span class="kwrd">void</span> DetachEvents(BeforeAndAfterTest form)
            {
                form.comboBox.SelectedIndexChanged -=
                    form.comboBox_SelectedIndexChanged;
            }

            <span class="kwrd">private</span> <span class="kwrd">static</span> <span class="kwrd">void</span> AttachEvents(BeforeAndAfterTest form)
            {
                <span class="rem">// Detach first to prevent accidental double subscription.</span>
                form.comboBox.SelectedIndexChanged -=
                    form.comboBox_SelectedIndexChanged;
                form.comboBox.SelectedIndexChanged +=
                    form.comboBox_SelectedIndexChanged;
            }
        }</div>
    }</pre>
<p>Finally, I don't want to leave out my new favorite language.&nbsp; Below is an implementation of the <em>BeforeAndAfterBlock</em> class in F# along with a simple test in the console.&nbsp; Not so functional, but it works.&nbsp; Perhaps there is a way to do this with computation expressions?</p><pre class="code"><span style="color: blue">open </span>System

<span style="color: blue">type </span>BeforeAndAfterBlock(fBefore, fAfter) =
  <span style="color: blue">do </span>fBefore()

  <span style="color: blue">interface </span>IDisposable <span style="color: blue">with
    member </span>x.Dispose() = fAfter()

<span style="color: blue">let </span>fBefore = <span style="color: blue">fun</span>() <span style="color: blue">-&gt; </span>printfn <span style="color: maroon">"Before"
</span><span style="color: blue">let </span>fAfter = <span style="color: blue">fun</span>() <span style="color: blue">-&gt; </span>printfn <span style="color: maroon">"And After"

</span><span style="color: blue">let </span>testBlockWithType() =
  <span style="color: blue">use </span>printMessageBlock = <span style="color: blue">new </span>BeforeAndAfterBlock(fBefore, fAfter)
  printfn <span style="color: maroon">"Middle"

</span>testBlockWithType()

ignore(Console.ReadLine())</pre>
<p>So that's it.&nbsp; A pretty cool and non-standard way of using <em>IDisposable</em> and the using statement.</p>