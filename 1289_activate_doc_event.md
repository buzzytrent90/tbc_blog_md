---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.689924'
original_url: https://thebuildingcoder.typepad.com/blog/1289_activate_doc_event.html
post_number: '1289'
reading_time_minutes: 5
series: general
slug: activate_doc_event
source_file: 1289_activate_doc_event.htm
tags:
- references
- revit-api
- vbnet
title: Open and Activate Document in Event Handler
word_count: 1009
---

### Open and Activate Document in Event Handler

I have been a busy beaver with the Revit API on the
[discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) in
the past few days, and on
[GitHub](https://github.com/jeremytammik) as well.

I want to take a closer look at two related issues that date back a little bit further:

- [Switching active documents is not allowed during API event handling](http://forums.autodesk.com/t5/revit-api/switching-active-documents-is-not-allowed-during-api-event/m-p/5498500)
- [Loading a RVT project file automatically on start-up](http://forums.autodesk.com/t5/revit-api/how-to-load-a-rvt-file-automatically-when-start-the-revit/m-p/5500878)

Let's discuss the first today and save the second for later.

Before getting to that, I also want to mention the Autodesk Foundation that is currently celebrating its one-year anniversary.

#### Autodesk Foundation

The Autodesk Foundation is the first foundation to focus investment exclusively on the people and organizations using design for impact.
**Impact design** is a growing trend among designers who are using their talents to create a better world.
The fields of public interest, social impact, community, humanitarian, and sustainable design—together making up impact design—are all on a critical growth path as more people use the power of design for positive social and environmental impact.

Here is a [three and a half minute video](https://www.youtube.com/watch?v=JV0HyXjpMi0) explaining more about the Autodesk Foundation and impact design and describing how:

- 15 impact design organizations were supported by the Autodesk Foundation. One said, “In less than a year, the Autodesk Foundation has become one of the most sophisticated foundations working in the field.”
- 2700 nonprofits were supported by employees and/or the Autodesk Foundation with funding, technology, and volunteer time.
- 1500 Autodeskers joined in making a positive impact at work, at home, and in the community.

Back to the Revit API...

#### Open and Activate a Document During an API Event

This query on
[switching active documents is not allowed during API event handling](http://forums.autodesk.com/t5/revit-api/switching-active-documents-is-not-allowed-during-api-event/m-p/5498500) was
raised by
[Luigi Trabacchin](https://github.com/LiFeleSs).

If you wish to fast-forward straight to the solution, it consists in using an external event and XAML in VB.NET, and is demonstrated by the
[VB.NET RevitOpenAndActivateDuringIdle sample on GitHub](https://github.com/LiFeleSs/RevitOpenAndActivateDuringIdle).

Here is the edited discussion leading up to this result:

**Question:**
This really self-explanatory exception is thrown when I try to do an Application.OpenandActivateDocument, the second time (when I have no active documents it works),

The point is the event I'm trying to do this from, in the application Idling event handler.

I really expected to be allowed to do that from there.

**Answer:**
Does this discussion on
[closing the active document](http://thebuildingcoder.typepad.com/blog/2012/12/closing-the-active-document.html) help
in any way?

**Response:**
Hello Jeremy I went through your blog thoroughly before posting and that article has been read.

I tried to close the document but I get another error, I think process start can help, but I dislike that option since I don't get any reference to the document ... maybe I can get through open documents and find the one with that filename...

First call works from second on it doesn't.

I made a solution
[RevitOpenAndActivateDuringIdle](https://github.com/LiFeleSs/RevitOpenAndActivateDuringIdle) to
investigate the problem with the community.

**Answer** by Arnošt Löbel:
Naturally, I have to step in, for I do not like Jeremy’s suggested solution all that much (and he knows it :-)

Although opening a new active document is not allowed from event handlers when there already is a document open in Revit, there may be an alternative. I believe we have implemented External Events so they do not share this restriction with events. Thus, for the user who otherwise used the Idling event, switching to an External Event could be a suitable solution. And it is both supported and safe.

Of course, just as you say, opening a DB document has no restrictions. You should be always able to do it in virtually any situations.

There is some help on using External Events in the SDK samples and Revit API help file.

**Response:**
I've found an example about that on Jeremy's blog, my source for inspiration (thanks Jeremy) and updated my solution to not use the Idling event but to raise my shiny new external event, and it works like a charm!

I didn't realize the external events purpose; I thought it was a way to leverage Idling handling...

So thanks for pointing me in the right direction!

I updated the
[GitHub repository](https://github.com/LiFeleSs/RevitOpenAndActivateDuringIdle),
if you want to include it in the samples...

**Answer:**
Congratulations on resolving your problem, and thank you for sharing the result.

I would love to share your sample with the rest of the Revit add-in developer community...

Could you be a bit more specific about what it actually demonstrates and how, now that all the issues you were encountering are resolved and you are clear about the use of external events?

**Response:**
Very good!
You can share my example as long as you want; in fact it is publicly available on
[GitHub](https://github.com/LiFeleSs/RevitOpenAndActivateDuringIdle).

The last version of it demonstrates how using an ExternalEvent solves getting the "Switching active documents is not allowed during API event handling" exception trying to open a document (or more than one) during a call from a command or an event such the Idling.

And also it demonstrates how to declare and call the external event.

The previous versions could be also quite useful to some Revit API users because it shows how to get a task executed during the Idling event, but needs some rearrangement because it was built to demonstrate the other issue.