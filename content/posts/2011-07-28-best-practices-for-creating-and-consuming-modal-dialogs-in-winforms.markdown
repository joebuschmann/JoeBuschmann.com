---
layout: post
title: Best Practices for Creating and Consuming Modal Dialogs in WinForms
date: '2011-07-28 19:37:55'
tags:
- net
- c
- winforms
---

This is an article I wrote a few years ago after seeing some bad coding practices around modal dialogs in WinForms.┬á Bad habits like explicitly closing the dialog form and using custom OK/Cancel flags irked me, so I came up with some best practices.┬á I hope you find them useful.

**Tip 1: Set the AcceptButton and CancelButton properties**

Windows Forms have two properties, *AcceptButton* and *CancelButton*, for determining what should be done when the user presses the Enter or Escape keys. You can set the *AcceptButton* property value to the name of an existing button on the form, normally an OK or Yes button, to fire that button's click event when the user presses the Enter key. Similarly, you can set the *CancelButton* property to an Cancel or No button to fire that button's click event when the user presses the Escape key.

![AcceptButton and CancelButton properties](http://media.joebuschmann.com/formproperties.png)

**Tip 2: Use a Button's DialogResult property when applicable**

The Button class has a property, *DialogResult*, that can be set to one of several values in the `System.Windows.Forms.DialogResult` enumeration. This value is returned by the `Form.ShowDialog()` method indicating the result of the modal dialog operation when the button is clicked. For example, you can set this property to `DialogResult.OK` for an OK button and `DialogResult.Cancel` for a Cancel button.

![DialogResult property](http://media.joebuschmann.com/formproperties2.png)

**Tip 3: Don't explicitly hide the form**

If a button's *DialogResult* property is set to any value besides `DialogResult.None`, there is no need to explicitly hide the form in the button's click event. The form will automatically close and control will return to the calling code. Setting the form's *DialogResult* property to `DialogResult.None` in the button's click event will prevent the form from closing. You can do this if validation fails, and you want to keep the form visible.

<script src="https://gist.github.com/joebuschmann/6a10c5813be82e46cc5a.js"></script>

**Tip 4: Employ the *using* statement**

Wrap the code that displays and processes a modal dialog window in a *using* block to ensure the window is disposed properly. This is a simple and obvious tip but one that is often overlooked.

<script src="https://gist.github.com/joebuschmann/4e5965df1c536682f72a.js"></script>

**Tip 5: Prefer overriding a form's OnLoad method**

From within subclasses of the Form class, prefer overriding the *OnLoad* method over attaching an event handler to the Load event. Of course, you should still call the base class's *OnLoad* method. You can use this method to process input from the calling code that is set between the time the constructor is executed and the window is displayed.

<script src="https://gist.github.com/joebuschmann/2ca78bd08ce7eb149055.js"></script>

**Tip 6: Override *Form.ProcessDialogKey()* to suppress the accept and/or cancel buttons**

There are situations where you may not want the accept or cancel button's code to execute on a key press event. For example, if a filter editor is displayed at the top of the window and you would like the filter to be applied when the user presses enter, you can override the `Form.ProcessDialogKey()` method and check to see if the accept button should be suppressed. Similarly, you can suppress the cancel button when the escape key is pressed.

<script src="https://gist.github.com/joebuschmann/3aaf386381926ad90cd3.js"></script>

An alternative is to suppress the accept and cancel buttons from within a custom user control. This is especially useful for controls that contain text editors where the return key is needed for adding line breaks, and the accept button should always be suppressed regardless of the containing form.

<script src="https://gist.github.com/joebuschmann/3d4608cee4566a89d4e2.js"></script>

**Bringing It All Together**

The following code samples demonstrates good and bad practices when coding modal dialog windows.

When displaying a modal dialog window...

<span style="color: #00ff00;">do this:</span>

<script src="https://gist.github.com/joebuschmann/c54d16c3cff101960f91.js"></script>

<span style="color: #ff0000;">don't do this:</span>

<script src="https://gist.github.com/joebuschmann/a8da72fcdaaea1847b9c.js"></script>

In the modal dialog form's OK and Cancel button click events...

<span style="color: #00ff00;">do this:</span>

<script src="https://gist.github.com/joebuschmann/caede8ad756c68832512.js"></script>

<span style="color: #ff0000;">don't do this:</span>

<script src="https://gist.github.com/joebuschmann/ca5b5147ef6b6391a73c.js"></script>
