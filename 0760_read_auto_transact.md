---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: documentation
optimization_date: '2025-12-11T11:44:14.536168'
original_url: https://thebuildingcoder.typepad.com/blog/0760_read_auto_transact.html
post_number: '0760'
reading_time_minutes: 3
series: general
slug: read_auto_transact
source_file: 0760_read_auto_transact.htm
tags:
- family
- revit-api
- transactions
- views
title: Read-only and Automatic Transaction Modes
word_count: 659
---

### Read-only and Automatic Transaction Modes

We have been discussing
[transactions](http://thebuildingcoder.typepad.com/blog/2012/04/scope-and-dispose-transactions.html) and
[their disposal](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html) quite
a bit recently, with lots of interesting input from Guy Robinson and Arnošt Löbel.

They just submitted
[comments](http://thebuildingcoder.typepad.com/blog/2012/05/edit-family-requires-no-transaction.html?cid=6a00e553e168978833016766068f67970b#comment-6a00e553e168978833016766068f67970b) on
[EditFamily requiring a non-modifiable document](http://thebuildingcoder.typepad.com/blog/2012/05/edit-family-requires-no-transaction.html) which
highlight another issue in this area worthwhile promoting to a post of its own to ensure that you take notice:

**Guy says:** ... maybe it's time to recommend using the Manual transaction mode only and the removal of Automatic mode.
This would pave the way for removing the transaction and regeneration attributes all together.

**Jeremy says:** I personally never use automatic transaction mode, but I do use both the manual and read-only ones.

It would actually be interesting to know the exact difference between an external command using read-only mode and one using manual without opening any transactions.

**Arnošt says:** I believe I said it publicly before, but probably not on this blog yet:

First of all, **Automatic** Transaction Mode is considered obsolete, and we (Autodesk.Revit) do not recommend using it for new external applications.
It only exists to ensure compatibility with older applications.
The Automatic mode has many limitations (for what an external command can or cannot do), while it has virtually no benefits whatsoever.
It is likely it will go away in some future release of Revit.

**Read-Only** transaction mode does two things (well, it's one thing, really, but I'll split it):

- For one, the command using it cannot have any transactions and may not call any methods that modify a document, directly or indirectly, or even temporarily.- Secondly, while this external command is being executed, nobody else is allowed to modify the document or have a transaction either; that includes Revit itself, but also applies to other applications that might be invoked through events (for example).

All this basically guarantees that during the entire execution of the external command the current models all remain unmodified.

**Guy sums up enthusiastically:** Yes, I've been a manual transaction person for a long time.

For various reasons, this enjoyable and thought provoking discussion has accelerated a process I have been slowly going through since the
R13 alpha's.
The richness of the R13 API effected a rethink, and issues such as transactions/exceptions, which I'll admit I hadn't changed my approach to for a very long time, are triggering more fundamental changes in attitude to the API.

I'd have to say compared to the R8 API I first played with a very long time ago, the speed, stability and functionality possible now is quite stunning.
And thinking why these scoping and other changes could be happening is making me rather excited.

With Win8/WinRT/iOS/mobile etc., though, apart from keeping it all logically in my brain it all makes for exciting times as a programmer, even if there aren't enough hours in the day ;-)

**Jeremy says:** Wow, yes, talking about cloud and mobile computing and enough hours in the day, have you looked at Kean's latest posts on
[creating a 3D](http://through-the-interface.typepad.com/through_the_interface/2012/04/creating-a-3d-viewer-for-our-apollonian-service-using-android-part-1.html)
[Android viewer](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-android-part-2.html),
[calling a web-service from Unity](http://through-the-interface.typepad.com/through_the_interface/2012/04/calling-a-web-service-from-a-unity3d-scene.html),
etc.?

Many thanks to Guy and Arnošt for this and all the other recent interesting discussions and insights!