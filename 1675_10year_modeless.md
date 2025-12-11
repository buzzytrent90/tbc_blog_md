---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.504430'
original_url: https://thebuildingcoder.typepad.com/blog/1675_10year_modeless.html
post_number: '1675'
reading_time_minutes: 4
series: general
slug: 10year_modeless
source_file: 1675_10year_modeless.md
tags:
- elements
- parameters
- revit-api
- sheets
- transactions
- views
- windows
title: 10year Modeless
word_count: 746
---

### Ten Years of TBC and Revit API with MVVM, WPF and WinForm
Today is The Building Coder's tenth birthday!
The first post was
a [warm welcome on August 22, 2008](http://thebuildingcoder.typepad.com/blog/2008/08/welcome.html).
Very many thanks to the entire community for all your support, interest, comments and above all numerous contributions over the years!
A suitable quote for the day:
*Out of clutter, find Simplicity.

From discord, find Harmony.

In the middle of difficulty lies Opportunity.

Albert Einstein – Three Rules of Work*

![Bumble bee in lotus flower](img/809_bee_in_lotus_580x500.jpg)
Today, let's pick up the recurring topic of accessing the Revit API from a modeless context.
It was discussed fruitfully in some depth in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [Revit API with WPF](https://forums.autodesk.com/t5/revit-api-forum/revit-api-with-wpf/m-p/8209618).
I'll skip most of the answer, because it is convoluted.
The most important aspect, though, is short and sweet:
[The Revit API cannot be used in a modeless context](http://thebuildingcoder.typepad.com/blog/2015/12/external-event-and-10-year-forum-anniversary.html#2).
It can only be used in a valid Revit API context.
A valid Revit API context is provided within the handlers executed by Revit call-backs, and nowhere else.
For instance,
an [external command](http://www.revitapidocs.com/2018/ad99887e-db50-bf8f-e4e6-2fb86082b5fb.htm) defines
an `Execute` methods and registers itself with Revit.
Revit calls that method and supplies a valid Revit API context to it via its `ExternalCommandData` input argument.
A Windows form can be displayed either using `Show` or `ShowDialog`.
If you use `Show`, it becomes a modeless dialogue and all interaction which it is outside of any valid Revit API context.
If you use `ShowDialog`, it becomes modal and remains in the same thread that launched it, possibly inheriting a valid Revit API context.
If you happen to be in a modeless context, you can implement a solution to access a valid Revit API context by implementing
an [external event](http://www.revitapidocs.com/2018/05089477-4612-35b2-81a2-89c4f44370ea.htm) and calling that.
Similarly to the external command, it also defines an `Execute` methods and registers itself with Revit.
Revit calls that method and supplies a valid Revit API context to it when it is prompted by an external trigger.
The official approach for implementing an external event is demonstrated by a Revit SDK sample:
- ModelessDialog/ModelessForm_ExternalEvent
It may require some redesign of your application logic to use.
Ryan Singleton added some good real-world advice to this:
To add my experience to what Jeremy said (you need External Events to make your transactions work modelessly), I have built a dockable dialog with WPF that used a lot of external events. Autodesk recommends that you try to use modal dialogs as much as you can instead of modeless. It will make your project much easier.
However, I can tell you that a fully functional suite of tools with external events is possible. But you need to be sure that the added complexity is worth it.
If you are going to have multiple external events, I strongly recommend that you look at the Revit SDK sample for External Events specifically. It shows how to use enums and a handler to swap out external events so that you do not have a sloppy mess of external events in your application.
You can also have a modeless dialog that, when you press a button to start a transaction, will open a modal window (progress bar or some other window), and that window will start the transaction instead. Once done, it closes, and the user feels like they are using a modeless context. A system like that would be much easier to maintain.
I hope this helps.
Here are some other recent threads raising the same question:
- [Revit Pick element from WinForm](https://stackoverflow.com/questions/51918495/revit-pick-element-from-winform)
- [Using MVVM and WPF with transaction and `doc.regenerate`](https://forums.autodesk.com/t5/revit-api-forum/using-mvvm-amp-wpf-with-transaction-amp-doc-regenerate/m-p/8207716)
Many other discussions and solutions are listed by The Building Coder in the topic group
on [Idling and external events for modeless access and driving Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28).