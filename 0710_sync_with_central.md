---
post_number: "0710"
title: "Synchronize with Central"
slug: "sync_with_central"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'windows']
source_file: "0710_sync_with_central.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0710_sync_with_central.html"
---

### Synchronize with Central

Developers have often asked how to programmatically synchronise with central, which is currently not supported by the Revit API.

I discussed this issue once again recently with Erik Eriksson of
[White Arkitketer AB](http://www.white.se),
and he tested and verified that yet another risky workaround can be used to achieve this, as always
[at your own peril](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html#3).

**Question:** I have a worksharing project in which I am trying to open a file, make changes, synchronize the changes to the central file and then close the file.
I can complete all the tasks needed except synchronization part.
Is that not possible through the API?
I haven't found anything regarding this in the API manuals for Revit 2011 and 2012.

Is there a workaround for this?
Our project is a very complex hospital of 370 000 m2 and we have 17 different project files.
Making changes to these through the API would be a really powerful tool for us and we must be able to synchronize the changes to the central file quickly since there are 90 people working in these 17 files.

Any thoughts are welcome.
I've tried opening the central files, making changes, saving these and closing.
But that doesn't relinquish the elements changed.

I looked at your two year old post on auto-confirming a save dialogue by
[subscribing to the DialogboxShowingEvent](http://thebuildingcoder.typepad.com/blog/2009/06/autoconfirm-save-using-dialogboxshowing-event.html).

Could this also be used to auto-confirm the synchronize with central dialog box that I can bring up using the API?

**Answer:** There are several different possible ways to suppress unwanted dialogues programmatically in Revit, both using the Revit and the Windows API:

The old method that was already available in Revit 2010 was to
[use the DialogBoxShowing event](http://thebuildingcoder.typepad.com/blog/2009/06/autoconfirm-save-using-dialogboxshowing-event.html), as you discovered, which can be used to dismiss a dialogue programmatically.

The Revit 2011 API also includes a comprehensive
[failure processing API](http://thebuildingcoder.typepad.com/blog/2010/04/failure-api.html) which
can used to do much more than reacting to the DialogBoxShowing event.

If all else fails and you just want to get rid of the dialogue box by any means, you can also use the Windows API to detect when it is displayed and dismiss it using my
[dialogue clicker application](http://thebuildingcoder.typepad.com/blog/2009/10/dismiss-dialogue-using-windows-api.html).

In all three cases, you have to invest some research of your own to discover (in the debugger) what exact conditions you can use to reliably determine that the dialogue that is popping up really is the one you are interested in.
The approaches for that are described in the blog posts.

To optimise performance and minimise the impact and complexity, you might also want to set up your event handlers so that they are only active for a limited period of time, when you are expecting the unwanted dialogue to be displayed, and then remove them again after dismissing it.

**Response:** Success!
I used your solution to close the active document by
[executing a SendKeys call](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html) in
a separate thread.

In principle I am doing the same thing and triggering the synchronisation using
```csharp
SendKeys.SendWait("{F10}");
SendKeys.SendWait("3");
SendKeys.SendWait("{ENTER}");
```

The question is: what do you think about this?

I am only using the official UI functionality, and it should work irrespectively of whether a user or my SendKeys calls are driving it, shouldn't it?

**Answer:** Well, you should definitely pay close attention to the warnings of Arnošt Löbel, voiced in various blog posts, including the
[disclaimer](http://thebuildingcoder.typepad.com/blog/2010/10/closing-the-active-document-and-why-not-to.html#3) on
the very post you found, and yet again in all clarity in his class on this topic at Autodesk University 2011,
[**CP5381**](http://au.autodesk.com/?nd=event_class&session_id=9879&jid=1763185),
on asynchronous interactions and managing modeless UI, which would be well worth a blog post all on its own, at least.

Maybe using
[UI Automation](http://thebuildingcoder.typepad.com/blog/automation) instead
might possibly be a slightly safer bet, since I heard rumours that the addressee of the SendKeys messages may sometimes get confused.

**Response:** Yes, indeed.
In order to make this as safe as possible, I execute it during an idling event to make sure that the document really is ready.
I also subscribe to the DocumentSynchronizedWithCentral to make sure that the document was synchronized.

I attended Arnošt's class during AU so I am very cautious with non-supported API-calls.