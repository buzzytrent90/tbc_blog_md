---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.904199'
original_url: https://thebuildingcoder.typepad.com/blog/1383_external_event_10_year.html
post_number: '1383'
reading_time_minutes: 6
series: general
slug: external_event_10_year
source_file: 1383_external_event_10_year.md
tags:
- csharp
- elements
- geometry
- python
- revit-api
- rooms
- sheets
- transactions
- views
title: External Event 10 Year
word_count: 1114
---

### External Events and 10 Year Forum Anniversary
Ten years ago today, on December 9, 2005, Jim Quanci posted
the [first thread](http://forums.autodesk.com/t5/revit-api/getting-started-with-the-revit-net-api/m-p/1506776) on
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160).
![Ten year anniversary](img/ten_years.jpeg)
![Revitalizer](img/revitalizer.png)
This was pointed out by Revitalizer, one of the forum's chief contributors.
Thank you very much for this, Revitalizer, and thank you even more for all the good advice and numerous solutions you have provided to the community!
I celebrated this momentous event by revamping The Building Coder navigation bar at the bottom of this page, adding direct links to the [forum](http://forums.autodesk.com/t5/revit-api/bd-p/160),
[online Revit API help](http://revitapisearch.com), and
[developer guide](http://help.autodesk.com/view/RVT/2016/ENU/?guid=GUID-F0A122E0-E556-4D0D-9D0F-7E72A9315A42).
The number of my forum posts right now is 1366, so it has not yet quite overtaken the number of blog posts, 1383, but it certainly will really soon.
#### Use of the Revit API Requires a Valid Context
One of the most frequently raised questions around the Revit API
is [how to drive Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28),
e.g., from a separate thread, a modeless dialogue, or a stand-alone executable.
As an indication of this, The Building Coder defines four different related categories,
[events](http://thebuildingcoder.typepad.com/blog/events),
[external](http://thebuildingcoder.typepad.com/blog/external),
[Idling](http://thebuildingcoder.typepad.com/blog/idling) and
[modeless](http://thebuildingcoder.typepad.com/blog/modeless).
It was raised again just yesterday by Sizy458 in the query on how
to [call Revit's API from an external thread]:
\*\*Question:\*\* I created a transaction in the event handler for `ComponentManager_UIElementActivated` raised by a ribbon button.
That throws an exception saying 'Starting a transaction from an external application running outside of API context is not allowed'.
Have you a solution?
\*\*Answer by Arnošt Löbel:\*\* As you have deduced and as the exception message indicates, calling into the Revit API from outside threads (which include calls from modeless dialogs, for they too run on their own threads) is not permitted in Revit. This policy applies to every API method (except a very few). Some of them, e.g., transactions, enforce this rule by throwing an exception.
That said, there is a solution for those who want to communicate with the Revit API from their modeless dialogs or other external work threads. The key is that all communication between the outside thread and Revit API must be done either inside a handler of an Idling event or an External Event. Both are Revit API features and are described in the SDK documentation. The SDK even includes a sample of a Modeless Dialog, in which both the above mentioned approaches are demonstrated - one for the Idling event, one for an External Event. Please look in the SDK folder ModelessDialog for more information. Also, since this is a very frequently asked and answered question, it may be to your benefit to search the web or even this very forum for more details yourself.
\*\*Answer by Jeremy:\*\* This topic has probably come up and been answered a few hundred times in the past.
The short answer is: it is completely impossible to drive Revit or make any use whatsoever of the Revit API from an external thread.
The Revit API can only be used within a valid Revit API context.
Such a context is only available within callback methods that you provide to Revit and that Revit calls back when it is ready to accept your input.
A typical example of such a callback method is the Execute method of an external command.
The Building Coder provides an overview of the main discussions and explanations related
to [Idling and external events for modeless access and driving Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28).
You might want to start exploring the links provided there beginning at the end.
#### Driving Revit from Outside through an External Event
Size458 replied just a few hours later with a perfect summary of a perfect solution:
\*\*Response:\*\* That resolves my problem.
I used an external event like this:

```
  IExternalEventHandler handler_event
    = new ExternalEventMy();

  ExternalEvent exEvent
    = ExternalEvent.Create( handler_event );
```

The external event handler class implements the actions I need to execute in a valid Revit API context:

```
  class ExternalEventMy : IExternalEventHandler
  {
    public void Execute(UIApplication uiapp)
    {
      UIDocument uidoc = uiapp.ActiveDocument;
      if( null == uidoc )
      {
        return; // no document, nothing to do
      }
      Document doc = uidoc.Document;
      using( Transaction tx = new Transaction(doc))
      {
        tx.Start("MyEvent");
        // Action within valid Revit API context thread
        tx.Commit();
      }
    }
    public string GetName()
    {
      return "my event";
    }
  }
```

I can raise the external event from another thread like this:

```
  exEvent.Raise();
```

Many thanks Sizy458 for sharing this nice and succinct example of how to address this need.
#### Addendum – Idling versus External Event
I added this a day later, prompted
by [Michael's comment](http://thebuildingcoder.typepad.com/blog/2013/12/replacing-an-idling-event-handler-by-an-external-event.html#comment-2402180196):
\*\*Question:\*\* I have an Idling function that checks to see if Revit links have been updated so that I can prompt the user to reload the link if its newer. It works great when there are only a couple of links. More than that and it appears the Idling event will stop and start over, in which case the first idling event never finished so I get a repeating dialog. Do you think this is a problem with the Idling event that it could end and fire again without completing the rest of the code in it? If so I can try changing it to an external event. Thanks.
\*\*Answer:\*\* In general, I recommend not to use the Idling event at all, except for a one-off call.
If you want to use it for continuous polling, it gives you absolutely no control over the frequency.
Revit will call it as often as it can, hogging the entire CPU time and bringing performance down.
An external event will give you much more control and flexibility.
Therefore, I definitely recommend you switch to an external event instead of using Idling.
That is why I did this myself, as described in the description
on [replacing an Idling event handler by an external event](http://thebuildingcoder.typepad.com/blog/2013/12/replacing-an-idling-event-handler-by-an-external-event.html).
You can explore
the [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) for more details on that.