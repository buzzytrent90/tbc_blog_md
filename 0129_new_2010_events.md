---
post_number: "0129"
title: "New 2010 Events"
slug: "new_2010_events"
author: "Jeremy Tammik"
tags: ['revit-api', 'transactions', 'views']
source_file: "0129_new_2010_events.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0129_new_2010_events.html"
---

### New 2010 Events

The Event API in Revit has been completely rewritten in 2010 to be compliant to the .NET event standard.
The old events are still available but have been deprecated.
[Matt Mason](http://cadappdev.blogspot.com)
already posted an overview of some of the new
[event handling](http://cadappdev.blogspot.com/2009/03/whats-new-in-revit-2010-api-events.html)
features.
Transitioning from the old events to the new ones can include some pitfalls, and one has now been uncovered.

**Question:**
My application connects to a database when a project is opened.
It uses the following code which is called when a document is opened:

```
void revitApp_OnDocumentOpened( Document doc )
{
  try {
    doc.BeginTransaction();
    // . . .
    doc.EndTransaction();
  }
  catch (Exception ex)
  {
    MessageBox.Show( ex.Message );
  }
}
```

In Revit 2009, this occasionally produces the following error message when doc.BeginTransaction() is called, especially when work sharing is enabled in a project:
*"Attempted to read or write protected memory. This is often an indication that other memory is corrupt."*
This worked fine in the past. What is going wrong now?

**Answer:**
The OnDocumentOpened event is one of the old pre-2010 events.
In these, an API transaction can be started, but it is not guaranteed whether it will be successful or not.
This is one reason why the old events have been declared obsolete in 2010.
The Revit development team recommends moving on to the new 2010 events instead.

In this case, you can use the application.DocumentOpened event.
In this new event, you can modify the document directly without having to call BeginTransaction to open your own transaction.
The transaction required has already been created by the event framework before calling the event handler, so the document is immediately modifiable.
Thus, the calls to begin, end and abort a transaction are not allowed in the new event handlers and an InvalidOperationException is thrown if one of them is called during the event.
Here is what the Revit API documentation on the DocumentOpened event has to say about this:

This event is raised immediately after Revit has finished opening a document. It is raised even when document opening failed or was cancelled (during DocumentOpening event).

Check the 'Status' field in events argument to see whether the action itself was successful or not.

This event is not cancellable, for the process of opening document has already been finished.

If the action was successful, the document may be modified during this event.
A transaction is already created at the time of the event and will be automatically closed at the end of the event.
There is no need to create another transaction during this event and it is, in fact, prohibited.

Please be aware that the following API functions are not available for the current document during this event:

- Close()- BeginTransaction()- AbortTransaction()- EndTransaction()- Save()- SaveAs()

Exception InvalidOperationException will be thrown if any of the above methods is called during this event.

The DocumentOpened event is meant to replace the older OnDocumentOpened event, which is now Obsolete.

Many thanks to Saikat Bhattacharya for handling this case!

#### Revit 2010 API Webcast

Don't forget to register for the upcoming
[Revit 2010 API webcast](http://www.adskconsulting.com/adn/cs/api_course_sched.php)
Wednesday next week, April 29 2009.
The course name is Revit API, the language is English, and the location is 'Webcast'.