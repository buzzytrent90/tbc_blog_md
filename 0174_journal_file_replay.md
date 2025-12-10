---
post_number: "0174"
title: "Journal File Replay"
slug: "journal_file_replay"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'levels', 'parameters', 'python', 'revit-api', 'transactions', 'windows']
source_file: "0174_journal_file_replay.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0174_journal_file_replay.html"
---

### Journal File Replay

In spite of pretty mixed weather I went on a climb in the alps with my friend Robert on Saturday, the route
[Kurze Kombination](http://www.bergtour.ch/tourenfuehrer/routen/id/943)
(4c) on the mountain
[Schmalstöckli](http://www.bergtour.ch/tourenfuehrer/bilder/id/1047)
(2012 m) close to the Lidernenen Hütte.

![Schmalstöckli mountain](img/schmalstoeckli.jpg)

Here I am after the climb, just below the summit:

![Jeremy near Schmalstöckli summit](img/jeremy_near_schmalstoeckli_summit.jpg)

And this is Robert on the summit itself:

![Robert on Schmalstöckli summit](img/robert_on_schmalstoeckli_summit.jpg)

We would made another climb on the same mountain if the weather had not been pretty grey and moist.
Anyway, now it is Monday morning, and I am back again to Revit programming.

We mentioned the Revit journal file repeatedly in previous posts, but a basic introduction to this topic was still missing.
There is a good reason for this, because Autodesk does not provide any official support for the usage of journal files as a means of automating a portion of the design or production process.
The only official support for journal files is as a replay mechanism for testing or reporting issues.
One can make use of the journaling mechanism totally at one's own risk, and there will almost certainly be changes, incompatibilities between versions, and other problems.

The testing and reporting mechanism provided by the journaling mechanism is fully accessible to Revit add-ins as well.
This access is demonstrated by the Revit SDK Journaling sample.
Anyone interested in making use of journaling for any purpose whatsoever should have a good look at this sample in order to understand the mechanism in more depth.

Being unsupported, the journaling mechanism should be used only as a last resort.
One example of a preferable approach for some tasks is to
[prompt the user](http://thebuildingcoder.typepad.com/blog/2009/07/prompt-the-user-for-pinning-and-ifc-export.html)
instead.

Still, with some imagination and due to the lack of automation support for
[driving Revit from outside](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html)
or
[invoking Revit commands](http://thebuildingcoder.typepad.com/blog/2009/05/vb-samples-and-other-questions.html#1)
such as the IFC export programmatically, it does sometimes offer interesting possibilities.

All the uses mentioned above are related to driving Revit by replaying an existing or an automatically generated journal file.
Another way of using the journal file is to read out its contents dynamically during an on-going session to find out what is going on.
I mentioned an idea to
[determine the last Revit command](http://thebuildingcoder.typepad.com/blog/2009/06/volume-computation-enable.html?cid=6a00e553e1689788330115708fd987970c#comment-6a00e553e1689788330115708fd987970c)
from it, and Greg Wesner discussed
[getting the journal file path](http://thebuildingcoder.typepad.com/blog/2009/02/getting-the-journal-file-path.html)
for other monitoring purposes.

Here are some recent questions on this topic, one of which lead to me finally putting together an example and proving to myself that a journal file can be used pretty reliably to drive a completely encapsulated sequence of commands, including Revit start up and shutdown. But first, let us look at a negative result.

**Question:**
Is it possible to play back a Revit journal file while Revit is already running and a document is already opened?

**Answer:**
Sorry, I am not aware of any way to play back a journal file in the middle of a running session, with an opened document.
The only way I know of using journal files for programmatic purposes is to include the entire sequence of starting up Revit, opening the document, executing the desired action, and closing down Revit again within the journal file execution.
You may possibly omit the closing down part, in which case Revit will be left open, if you are willing to give up control to the user at that point.

Here is the question that led me to test running Revit from a journal file and including steps in it which provide me API access for additional control during the replay process:

**Question:**
Is it possible to use the Revit API to create a new family within the OnStartup method of an external application?

**Answer:**
The argument provided to OnStartup is a ControlledApplication instance.
This object provides methods to manipulate the ribbon, access the shared parameters file, access some application level properties and set up application level event handler.

It does not include access to the documents.
That access is provided by the Documents collection on the Application class, which is provided as an argument to external commands, which are only active after OnStartup has completed and a document has been opened.

If you wish to start up Revit and immediately create a new family document, you can do so once manually and save the journal file that is generated by this interaction.
The journal file can then later be used to restart Revit and re-execute this process.

If you wish to continue working programmatically with the new family document, you can set up an event handler for the DocumentCreated or DocumentOpened notification.

Some notes on external applications and document event handling are provided by the blog entries:

- [External application OnStartup implementation in VB](http://thebuildingcoder.typepad.com/blog/2009/05/vb-samples-and-other-questions.html#3).
- [External application implementing automatic synchronisation using document save and close events](http://thebuildingcoder.typepad.com/blog/2009/01/barcelona-questions.html).
- [Transactions and document modification in a DocumentOpened event handler](http://thebuildingcoder.typepad.com/blog/2009/04/new-2010-events.html).
- [Document modification in the DocumentClosing event handler](http://thebuildingcoder.typepad.com/blog/2009/05/document-modification-in-an-event.html).
- [Programmatic handling of dialogue box events](http://thebuildingcoder.typepad.com/blog/2009/06/autoconfirm-save-using-dialogboxshowing-event.html).

Following this suggestion to implement an external application reacting to an event to programmatically process a newly created family file created by driving Revit from a journal file led to the following new question:

**Question:**
I have created a journal file that opens a family document on Revit start up.
However, it issues an error when calling my external application's OnStartup method saying "The journal file could not be run to completion."

**Answer:**
I cannot reproduce the problem you describe.
In my experience, the message that the journal file could not run to completion is due to other problems.
I do not believe it is caused by the event handler registration.
To test this, I implemented a new external application AutoExecuteOnOpen which does what you say, registering event handlers for the DocumentCreated and DocumentOpened events in the OnStartup method.
Both of these simply print a message to the Visual Studio debug output window:

```python
public IExternalApplication.Result OnStartup(
  ControlledApplication a )
{
  a.DocumentCreated
    += new EventHandler<DocumentCreatedEventArgs>(
      a\_DocumentCreated );

  a.DocumentOpened
    += new EventHandler<DocumentOpenedEventArgs>(
      a\_DocumentOpened );

  return IExternalApplication.Result.Succeeded;
}

public IExternalApplication.Result OnShutdown(
  ControlledApplication a )
{
  a.DocumentCreated
    -= new EventHandler<DocumentCreatedEventArgs>(
      a\_DocumentCreated );

  a.DocumentOpened
    -= new EventHandler<DocumentOpenedEventArgs>(
      a\_DocumentOpened );

  return IExternalApplication.Result.Succeeded;
}

void a\_DocumentCreated( object sender, Autodesk.Revit.Events.DocumentCreatedEventArgs e )
{
  Debug.Print( "DocumentCreated" );
}

void a\_DocumentOpened( object sender, DocumentOpenedEventArgs e )
{
  Debug.Print( "DocumentOpened" );
}
```

I registered this external application in Revit.ini.

I then ran through the following sequence and saved the resulting journal file:

- Start up Revit.- Open a family document.- Close the family document.- Shut down Revit.

These steps resulted in a new journal file:

```
C:\Program Files\Autodesk Revit Architecture 2010\Journals\journal.0180.txt
```

I saved this journal file for replay and had no problem rerunning it.
It executes all the following expected steps without problems:

- Start up Revit.- Load the external application.- Open a family document.- Fire the DocumentOpened event.- Print the message.- Close the family document.- Shut down Revit.

In the first attempt at implementing my external application event handlers, I displayed a message box.
This did not work, at least not when run from the debugger, because clicking OK in the message box had no effect, so the journal file execution was blocked at that point.
Replacing the message box by a simple Debug.Print statement solved that problem.
This shows that the entire execution process is very sensitive and picky, though.

Here is a copy of my
[journal file](zip/journal_file.txt),
and here the entire Visual Studio solution
[AutoExecuteOnOpen](zip/AutoExecuteOnOpen.zip).