---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.693667'
original_url: https://thebuildingcoder.typepad.com/blog/0831_dispose.html
post_number: 0831
reading_time_minutes: 3
series: general
slug: dispose
source_file: 0831_dispose.htm
tags:
- elements
- filtering
- parameters
- revit-api
- transactions
title: Disposal of Revit API Objects
word_count: 534
---

﻿

### Disposal of Revit API Objects

KR raised a very valid and
[important](http://thebuildingcoder.typepad.com/blog/2012/09/exporting-parameter-data-to-excel.html?cid=6a00e553e168978833017c3204cc10970b#comment-6a00e553e168978833017c3204cc10970b)
[question](http://thebuildingcoder.typepad.com/blog/2012/09/exporting-parameter-data-to-excel.html?cid=6a00e553e168978833017c3204cd86970b#comment-6a00e553e168978833017c3204cd86970b)
in the discussion of
[exporting parameters to Excel](http://thebuildingcoder.typepad.com/blog/2012/09/exporting-parameter-data-to-excel.html):

**Question:** It would be great if you could clarify in broad terms when to dispose of Revit API objects.

You underline the importance of
[disposing of Transaction objects](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html).

Many objects in the Revit API implement IDisposable, for example FilteredElementCollector, DB.Document, etc.

If I create a new FilteredElementCollector within the confines of Revit API, e.g. in an external command, Idling event handler, external event etc., do I still have to dispose of this object?

The rule of thumb in the .NET world is: 'Always dispose of all objects implementing IDisposable that you instantiated'.

**Answer by Arnošt Löbel:** You are absolutely correct – it is advisable to explicitly dispose every disposable object, either by calling its Dispose method, deleting it (in C++), or scoping it in a 'using' block.
However, it is not absolutely necessary in the managed environment, and there should normally be no harm if you do not do so.
The Garbage Collector (GC, for short) will collect the abandoned objects and everything should be just fine, memory-wise.

It might be a little problematic to depend only on the GC if there is 'something' that must be done by the object destructor or finaliser, and that 'something' is timely essential, meaning it should happen is some semi-predetermined time rather than randomly when the GC is triggered.
That is why, in my opinion, it is generally recommended to explicitly dispose of all disposable objects.
Again, it is not over-important for all objects; I would not bother with filtered element collectors, for example.

Transactions (and T-Groups, S-Transaction, Edit Modes, etc.), are different.
While it is no more (nor less) important to **dispose** of them explicitly and in a timely fashion, it is **imperative** that they are properly **'finished'**.
The 'finishing' may mean something different to each of these objects; for example – it might be either Committing or Rolling Back for transactions.
This 'finishing' is in fact the part that 'must' be done soon enough, most importantly before the API client (external command, event, etc.) terminates.
Since the 'finishing' procedure is automatically invoked from the object destructor, the disposing action indirectly 'finishes' the object as well.
This is the reason why it is important to scope transactions in a 'using' block; because 'using' does not let the object leave the scope without disposing (and finishing) it, even in the case of exceptions.

Let me repeat this in yet other words: if your application can guarantee that Commit or RollBack is always called for all transactions that have been Started, it is not necessary to dispose the transaction objects, either explicitly (dispose, delete) or implicitly (using).
However, the nice thing about 'using' blocks is that they automatically guarantee this.