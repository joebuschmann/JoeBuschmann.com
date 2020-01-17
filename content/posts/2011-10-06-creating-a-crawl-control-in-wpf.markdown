---
layout: post
title: Creating a Crawl Control in WPF
date: '2011-10-06 19:45:09'
tags:
- c
- wpf
- xaml
---

<p>A few weeks ago I was watching CNN when an interesting blurb scrolled by on the crawl at the bottom of the screen.&nbsp; Normally, the crawl is not something I notice, but I thought to myself, ΓÇ£I wonder if I can implement one in WPF.ΓÇ¥&nbsp; I had never done any animations before in WPF, and this seemed like a good way to start.</p> <p>So I gave it a shot.</p> <p>My initial instinct was to bring up Google and see if anyone else had done it.&nbsp; I stopped myself because this was a good learning opportunity.&nbsp; A chance to put my nascent XAML skills to the test.&nbsp; Afterward I could research the topic and see what I did or did not do right.</p> <p>My first challenge was to come up with a good visual layout that would enable the basic scrolling animation.&nbsp; I needed a panel of some type and a control to hold the scrolling text.&nbsp; The text would come from a bindable list so naturally an ItemsControl was the best choice.&nbsp; It would form the banner that would scroll across the screen.</p> <p>The panel was a little tougher.&nbsp; In WinForms, I would update the Left property to increasingly negative values to scroll it to the left; however, WPF framework elements do not have a Left property.&nbsp; WPFΓÇÖs control layout resembles HTML more than WinForms.&nbsp; Enter the Canvas control.&nbsp; It enables child elements to be positioned using coordinates, and I could animate the Canvas.Left attached property to scroll the banner.</p> <p>The complete control template is listed below.</p><pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">ControlTemplate </span><span style="color: red">TargetType</span><span style="color: blue">="{</span><span style="color: #a31515">x</span><span style="color: blue">:</span><span style="color: #a31515">Type </span><span style="color: red">Controls</span><span style="color: blue">:</span><span style="color: red">CrawlList</span><span style="color: blue">}"&gt;
    &lt;</span><span style="color: #a31515">Border </span><span style="color: red">Background</span><span style="color: blue">="{</span><span style="color: #a31515">TemplateBinding </span><span style="color: red">Background</span><span style="color: blue">}"
            </span><span style="color: red">BorderBrush</span><span style="color: blue">="{</span><span style="color: #a31515">TemplateBinding </span><span style="color: red">BorderBrush</span><span style="color: blue">}"
            </span><span style="color: red">BorderThickness</span><span style="color: blue">="{</span><span style="color: #a31515">TemplateBinding </span><span style="color: red">BorderThickness</span><span style="color: blue">}"&gt;

        </span><span style="color: green">&lt;!-- Use a Canvas as the parent panel to take advantage of absolute
             positioning which makes the animation easier. --&gt;
        </span><span style="color: blue">&lt;</span><span style="color: #a31515">Canvas </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Name</span><span style="color: blue">="crawlCanvas" </span><span style="color: red">VerticalAlignment</span><span style="color: blue">="Stretch"
                </span><span style="color: red">HorizontalAlignment</span><span style="color: blue">="Stretch"&gt;

            </span><span style="color: green">&lt;!-- An ItemsControl forms the scrolling banner. --&gt;
            </span><span style="color: blue">&lt;</span><span style="color: #a31515">ItemsControl </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Name</span><span style="color: blue">="crawlItems"
                          </span><span style="color: red">ItemsSource</span><span style="color: blue">="{</span><span style="color: #a31515">TemplateBinding </span><span style="color: red">ItemsSource</span><span style="color: blue">}"
                          </span><span style="color: red">Canvas.Top</span><span style="color: blue">="0"
                          </span><span style="color: red">Canvas.Left</span><span style="color: blue">="{</span><span style="color: #a31515">TemplateBinding </span><span style="color: red">Left</span><span style="color: blue">}"
                          </span><span style="color: red">Canvas.Bottom</span><span style="color: blue">="{</span><span style="color: #a31515">Binding </span><span style="color: red">Width</span><span style="color: blue">,
                                          </span><span style="color: red">ElementName</span><span style="color: blue">=crawlCanvas}"
                          </span><span style="color: red">Canvas.Right</span><span style="color: blue">="{</span><span style="color: #a31515">Binding </span><span style="color: red">Height</span><span style="color: blue">,
                                         </span><span style="color: red">ElementName</span><span style="color: blue">=crawlCanvas}"&gt;

                </span><span style="color: green">&lt;!-- The default item template is a simple text block.
                     It can be updated to a different template using
                     CrawlList.ItemTemplate.--&gt;
                </span><span style="color: blue">&lt;</span><span style="color: #a31515">ItemsControl.ItemTemplate</span><span style="color: blue">&gt;
                    &lt;</span><span style="color: #a31515">DataTemplate</span><span style="color: blue">&gt;
                        &lt;</span><span style="color: #a31515">TextBlock </span><span style="color: red">Text</span><span style="color: blue">="{</span><span style="color: #a31515">Binding</span><span style="color: blue">}" /&gt;
                    &lt;/</span><span style="color: #a31515">DataTemplate</span><span style="color: blue">&gt;
                &lt;/</span><span style="color: #a31515">ItemsControl.ItemTemplate</span><span style="color: blue">&gt;

                </span><span style="color: green">&lt;!-- By default the ItemsPanel property contans a StackPanel
                     with a vertical orientation.  Replace it with a StackPanel
                     with a horizontal orientation. --&gt;
                </span><span style="color: blue">&lt;</span><span style="color: #a31515">ItemsControl.ItemsPanel</span><span style="color: blue">&gt;
                    &lt;</span><span style="color: #a31515">ItemsPanelTemplate</span><span style="color: blue">&gt;
                        &lt;</span><span style="color: #a31515">StackPanel </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Name</span><span style="color: blue">="crawlItemsPanel"
                                    </span><span style="color: red">Orientation</span><span style="color: blue">="Horizontal" /&gt;
                    &lt;/</span><span style="color: #a31515">ItemsPanelTemplate</span><span style="color: blue">&gt;
                &lt;/</span><span style="color: #a31515">ItemsControl.ItemsPanel</span><span style="color: blue">&gt;
            &lt;/</span><span style="color: #a31515">ItemsControl</span><span style="color: blue">&gt;
        &lt;/</span><span style="color: #a31515">Canvas</span><span style="color: blue">&gt;

    &lt;/</span><span style="color: #a31515">Border</span><span style="color: blue">&gt;
&lt;/</span><span style="color: #a31515">ControlTemplate</span><span style="color: blue">&gt;</span></pre>
<p>In the controlΓÇÖs template, the <em>Canvas.Left</em> attached property is bound to the CrawlControlΓÇÖs <em>Left</em> dependency property.&nbsp; This was my workaround because I could not find a way to animate <em>Canvas.Left</em> directly.&nbsp; It seems like a hack.&nbsp; Perhaps there is a better way?</p>
<p>After hooking up the <em>Left</em> property, all I had to do was animate it with an instance of the <em>DoubleAnimation</em> class.&nbsp; There is also a <em>CrawlAnimation</em> dependency property for custom animations.</p><pre class="code"><span style="color: blue">private void </span>StartCrawlAnimation()
{
    <span style="color: blue">if </span>((_banner != <span style="color: blue">null</span>) &amp;&amp; (_banner.ActualWidth &gt; 0))
    {
        <span style="color: #2b91af">DoubleAnimationBase </span>doubleAnimation =
            CrawlAnimation ?? BuildDefaultAnimation();
        BeginAnimation(LeftProperty, doubleAnimation);
    }
}

<span style="color: blue">private void </span>EndCrawlAnimation()
{
    BeginAnimation(LeftProperty, <span style="color: blue">null</span>);
}

<span style="color: blue">private </span><span style="color: #2b91af">DoubleAnimationBase </span>BuildDefaultAnimation()
{
    <span style="color: blue">double </span>bannerWidth = _banner.ActualWidth;
    <span style="color: blue">double </span>fromValue = _crawlCanvas.ActualWidth;
    <span style="color: blue">double </span>toValue = -1 * bannerWidth;
    <span style="color: blue">double </span>speed = CrawlSpeed;

    <span style="color: #2b91af">Duration </span>duration = <span style="color: blue">new </span><span style="color: #2b91af">Duration</span>(
        <span style="color: #2b91af">TimeSpan</span>.FromSeconds(bannerWidth / speed));

    <span style="color: blue">return new </span><span style="color: #2b91af">DoubleAnimation</span>(fromValue, toValue, duration)
        { RepeatBehavior = <span style="color: #2b91af">RepeatBehavior</span>.Forever };
}</pre>
<p>So there it is.&nbsp; My first animation.&nbsp; I was surprised at how easy WPF makes animations.</p>
<p>You can download the <a href="https://github.com/joebuschmann/Buschmann.Windows" target="_blank">complete source code</a>.&nbsp; The code behind is in CrawlList.cs, and the template is in CrawlListStyle.xaml.&nbsp; To see it in action, run the solution, and from the main window, click the ΓÇ£Crawl ListΓÇ¥ button to bring up a simple testing view.</p>