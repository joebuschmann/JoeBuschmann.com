---
layout: post
title: Ditch the Grids and Use DockPanels
date: '2012-03-12 19:58:25'
tags:
- c
- best-practices-2
- winforms
- wpf
---

<p>I have worked extensively with WinForms and WPF/Silverlight and noticed that docking/dock panels are not used that often.&#160; At least this is the case where I work.&#160; I find this surprising given their power and ease of use.</p>  <p>Using XAML, why would a programmer choose a complex Grid/StackPanel setup over a DockPanel?&#160; Or on the WinForms side, set the anchor properties for control rather than simplify things with the dock property?&#160; I see this stuff all the time.&#160; Perhaps it's a lack of knowledge.&#160; Maybe there is a performance penalty that I don't know about.</p>  <p>There are a couple of quirks with docking that confused me at first.&#160; In WinForms, when setting a control's Dock property to Fill, it may appear behind the other controls on the form rather than filling the remaining space.&#160; The fix is to right-click the control in the designer and choose &quot;Bring to Front&quot;.&#160; It will resize to fill the remaining space as one would expect.</p>  <p>In WPF/Silverlight, there is no Dock property.&#160; Instead the DockPanel control performs the same function.&#160; Child controls of the DockPanel set the attached Dock property to dock themselves.&#160; The values may be Left, Right, Top, and Bottom, but there is no Fill value.&#160; Instead the last child control in the XAML is assumed to be the fill control even if it sets a value explicitly.&#160; Or the fill panel can be turned off by adding <em>LastChildFill = &quot;False&quot;</em> on the DockPanel.&#160; This can be a little strange especially coming from WinForms.</p>  <p>These are the only two oddities I've come across; otherwise I have not seen any obvious performance or functionality issues.&#160; In my experience, docking works beautifully and is my preferred method for layout.</p>