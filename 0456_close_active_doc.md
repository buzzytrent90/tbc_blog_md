---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.2
content_type: code_example
optimization_date: '2025-12-11T11:44:13.976084'
original_url: https://thebuildingcoder.typepad.com/blog/0456_close_active_doc.html
post_number: '0456'
reading_time_minutes: 8
series: general
slug: close_active_doc
source_file: 0456_close_active_doc.htm
tags:
- csharp
- elements
- python
- revit-api
- transactions
- views
- windows
title: Closing the Active Document and Why Not To
word_count: 1536
---

### Closing the Active Document and Why Not To

Here is a question raised and solved by René Gerlach of
[CIDEON Software GmbH](http://www.cideon-software.com) in
collaboration with Joe Ye, and an explanation by Arnošt Löbel explaining the associated dangers:

**Question:** The documentation says that the overloads of the Document.Close method cannot close the active document.
How can this be handled anyway?
I really do want to programmatically close the active document.

**Answer:** Revit does not currently expose any API to close the active document programmatically.

I think an alternative possible method might be to send a Windows message to Revit to mimic the operation: click the big 'R' button and then select the 'Close' menu item.

We discussed some related issues in the past, e.g. sending messages to Revit, determining its window handle, and working with modeless dialogues:

- [Driving Revit from outside](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html)
  - [Revit window handle and modeless dialogues](http://thebuildingcoder.typepad.com/blog/2009/02/revit-window-handle-and-modeless-dialogues.html)
    - [Dialogue clicker, also sends Windows messages](http://thebuildingcoder.typepad.com/blog/2009/10/dismiss-dialogue-using-windows-api.html)
      - [Modeless pressure drop tool](http://thebuildingcoder.typepad.com/blog/2009/10/modeless-pressure-drop-tool.html)
        - [Automated testing and an overview of previous automation solutions](http://thebuildingcoder.typepad.com/blog/2009/12/au-and-automated-testing.html)
          - [Revit parent window](http://thebuildingcoder.typepad.com/blog/2010/06/revit-parent-window.html)
            - [Modeless loose connectors](http://thebuildingcoder.typepad.com/blog/2010/07/modeless-loose-connectors.html)

Please be aware that sending keystrokes to Revit in this way is not part of the Revit API and is not officially supported by Autodesk.
It is actually working around some safeguards built into the Revit API framework, so please note the
[disclaimer](#3) below.
The discussion below also explains why
[these kinds of workarounds are strongly discouraged](#2).

But first, here is the workaround that we later on so strongly discourage:

**Response:** I initially tried to simply call the SendWait method to send a Ctrl + F4 keystroke to Revit as you suggest from within the external command implementation Execute method, but without any success:
```csharp
  SendKeys.SendWait( "^{F4}" );
```

Ctrl + F4 is one possible keyboard shortcut to close the current document.

Since this simple attempt did not work, I next tried using it in a separate thread.
The threaded solution works fine.

Here is the full source code of a new Building Coder sample command CmdCloseDocument based on René's solution:
```python
[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
class CmdCloseDocument : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    Document pDoc = commandData.Application
      .ActiveUIDocument.Document;

    ThreadPool.QueueUserWorkItem(
      new WaitCallback( CloseDocProc ) );

    return Result.Succeeded;
  }

  static void CloseDocProc( object stateInfo )
  {
    try
    {
      // maybe we need some checks for the right
      // document, but this is a simple sample...

      SendKeys.SendWait( "^{F4}" );
    }
    catch( Exception ex )
    {
      Util.ErrorMsg( ex.Message );
    }
  }
}
```

Launching this command closes the current document, just as desired.
If it is the only open document, you end up in the Revit project overview screen.

Although this works well in our tests so far to close the activate document, we don't know whether this approach can be used to send shortcut keys to trigger other commands as well.

Many thanks to Joe and above all to René for this solution.

Here is
[version 2011.0.76.0](zip/bc_11_76.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.

#### Windows Message Workarounds Discouraged

Even if this method worked in Revit 2010...
No cheating allowed...
We just discussed a similar issue about sending a shortcut trying to launch a different Revit command and had less luck with that.
A workaround to launch the command by sending a Windows message worked fine in Revit 2010 but no longer does so in Revit 2011.

This seems to be a tricky issue.
Adding or calling Revit native commands in the Add-Ins tab is not a supported scenario using the current Revit API.

In spite of all our efforts, we have not been able to port some code that worked in Revit 2010 to Revit 2011.
We used to send the shortcut string in a different thread, just as described above, but this still does not activate the Revit command.
As we saw above, we can send the Ctrl + F4 shortcut to close the active document, but sending a standard Revit keyboard shortcut to launch a Revit command fails.

Here is an excerpt from a discussion with the development team about this topic and possible workarounds, answered by Arnošt Löbel:

**Question:** Before giving up completely on solving this issue, I would like to hear your comments on it:

Could you think of any other possible things to try to make a shortcut used from an external command successfully launch a built-in Revit command?
We tried to achieve this by passing a keyboard shortcut through a Windows message, but it did not display the context tab.

The same approach worked fine in Revit 2010.

We are aware that this is not an officially supported usage scenario.

**Answer:** From my point of view, all the attempts mentioned are very dangerous indeed.
I would definitely not recommend building a commercial solution based on such approach.
We do not claim and never have claimed to support sending shortcuts (or any other commands) to the main Revit window and I do not think we should do that in the future.
It is a wrong approach in my opinion.
What we want to do and what we will do is to implement 'the right way' to provide the features our users need.
If you need to use the area command, we will look into implementing it and exposing it in the Revit API.

The claims that something hacky 'worked' in 2010 but does not work anymore in 2011, or vice versa, are all irrelevant.
Naturally, a lot of things can be achieved with the .NET or Windows API, but the Revit API does not support any of that for good reasons.

By the way, closing the active document from an external command using Windows messages as described above is a very nice way to crash Revit, eventually.
It bypasses the transaction protection we have implemented in the external command framework, as well as the protection we have in event framework.

It is one thing if an individual somewhere wants to do something experimental in his or her company only, for personal use, so to speak.
Another thing altogether would be if a developer builds an application based on such a technique and sells it as part of a solution.
The major problem is that the solution may work sometimes but not at other times.
It is really hard to tell and depends on various factors, such as what other external applications might also be installed on the system.
When we say that we do not support it, it is only a half the fact; the truth is that we do not even have a safety features in place against tricks like that, therefore anything could happen from just crashing Revit to corrupting document(s).

I am rather surprised that we do not hear about crashes and problems caused by these keystroke messages more often (or maybe we do, but don't know what caused them).

**Question:** Yes, I completely agree with you.
Something similar to the AutoCAD API method sendCommandToExecute in the Revit API would be nice.
And in this case, allowing customization of ribbon in UI might eliminate the need for this in the first place.

Thank you for strongly confirming that this approach is not supported.
These various attempts often help make us aware of real developer needs, though.

**Answer:** I totally agree with you.
We want to hear about users' and developers' needs and we want to make them happy.
Unfortunately, the Revit API users often feel like they do not need to tell us about their requirements because they can 'implement' it 'easily' themselves.
While we do encourage out users to experiment, any experimenting has limits.

I do not think I would like to even see anything similar to sendCommandToExecute in the Revit API.
We would need a framework to support this workflow and I personally do not think it is necessary.
The goal is to implement the native Revit UI itself (with all its commands) simply utilizing the internal Revit API, and to also expose that as our public API.
That way, API clients can do whatever our UI does and do not need to use these dangerous and unnatural workarounds.

Many thanks to Arnošt for this clear and supportive answer!

#### Disclaimer

The Windows message approach described above is a risky workaround which you might resort to if you are in desperate need.
Please be aware that this is not an officially supported usage by Autodesk.
It is not guaranteed to work under any conditions or in any past, present or future releases of Revit whatsoever.
If you use this, you are doing so at your own risk.
Autodesk hopes to expose an appropriate API for this functionality in future.