---
post_number: "0400"
title: "Revit Parent Window"
slug: "revit_parent_window"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'python', 'revit-api', 'schedules', 'views', 'windows']
source_file: "0400_revit_parent_window.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0400_revit_parent_window.html"
---

### Revit Parent Window

Today is the last day of the Munich AEC
[DevLab](http://thebuildingcoder.typepad.com/blog/2010/03/devcamp-devlabs-and-updated-api-training-schedule.html#devlabs).
It has been a great week here, both meeting with developers to discuss their issues, find quite a few solutions, and connect again with many old friends here in the Munich Autodesk office.
One issue that came up repeatedly was also a topic in the
[Waltham DevLab](http://thebuildingcoder.typepad.com/blog/2010/06/devlab-and-birthday.html),
so it is well worth exploring here in more detail:

People have repeatedly asked how to properly hook up their .NET forms to the Revit main application window so that it is always displayed on top of Revit and reacts properly when Revit is minimised and restored.
The technique presented below works equally well for modal and modeless forms.
Here is an example of such a query:

**Question:** I am displaying a dialogue that I want to associate with Revit.
Is there a way to explicitly specify the Revit window with the call to the ShowDialog method so that my form is associated with the Revit window instance and session?

**Answer:** I believe that the simplest way to achieve what you need is the following:

1. Determine the Revit main application window handle.- Implement an IWin32Window wrapper class returning the window handle.- Provide the IWin32Window instance as an argument to the ShowDialog method.

Let us look at these steps one by one:

1. If you are inside a Revit add-in, e.g. an external command, so that your code is part of Revit, so to speak, you can access the HWND window handle of the main Revit application using the following code:
```csharp
  Process process = Process.GetCurrentProcess();

  IntPtr h = process.MainWindowHandle;
```

2. The Windows.Forms Form.Show and ShowDialog methods are overloaded. You can call them without an argument, or with an instance of an IWin32Window interface implementation. That interface requires you to implement the Handle method, which can return the window handle determined in step 1. Here is an example of such a wrapper class implementation, with its constructor taking an IntPtr argument representing a HWND window handle:
```python
public class JtWindowHandle : IWin32Window
{
  IntPtr \_hwnd;

  public JtWindowHandle( IntPtr h )
  {
    Debug.Assert( IntPtr.Zero != h,
      "expected non-null window handle" );

    \_hwnd = h;
  }

  public IntPtr Handle
  {
    get
    {
      return \_hwnd;
    }
  }
}
```

I presented an example of implementing and using such a wrapper class in the discussion on
[driving Revit from outside](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html).
It demonstrates an alternative but less reliable method based on FindWindow to determine the main Revit window handle.

3. Provide an instance of your IWin32Window class to your form's Show or ShowDialog method.
```csharp
  form.ShowDialog( new JtWindowHandle( h ) );
```

Now your dialogue will be correctly recognised as a child window of the main Revit application window:

- The dialogue remains on top of the Revit window when it is not minimised.- The dialogue is automatically minimised when Revit is minimised, and restored again when Revit is restored.

I am attaching my
[LooseConnectors](zip/loose_connectors_4.zip) sample
application that I recently created for the AEC DevCamp Revit MEP API presentation.
It makes use of the
[MEP element collector and connector retrieval](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html)
methods that I presented a few days ago and then displays a list of all unconnected connectors in a modeless dialogue box.
It ensures that it stays on top of the Revit window by passing in the Revit window handle to the Form.Show method via an IWin32Window wrapper class, thus making Revit its parent window.
This requires converting the Windows API HWND which is represented as an IntPtr in .NET to an IWin32Window instance which is the incarnation preferred in the .NET System.Windows.Forms environment.
I will return to a more in depth discussion of this sample and various other of its interesting features and implementation aspects soon.

Here is some response to suggestions similar to those above:

**Response:** Thank you for the tip on making my form a child of the Revit window.
I did not realize that the Show method could simply take an argument.
Your wrapper for the IWin32Window was very helpful.

I discovered one remaining problem with handling a double click in a tree view, and implemented a workaround for that.

I was invoking a routine to place a family instance from a double-click event on a tree view in my form. It turns out that almost anything else works perfectly but not double-click.
I call it from the single-click of a button, and smoothly drag in my instance.
Same thing for a single-click in the list, or a right-click.
It seems that something about the double-click is pulling the focus back to my form, probably some kind of timing issue.
Clearly a windows forms issue, and nothing to do with Revit.

For now, I have changed my interface so the user selects the appropriate item in the list and right-clicks, and everything runs smoothly.
I thought you might want to know about this in case someone else runs into a similar glitch and can save some aggravation.
In the long run, I may want to figure out how to make it work with a double-click, but for the moment, I will move on to the many other challenges.