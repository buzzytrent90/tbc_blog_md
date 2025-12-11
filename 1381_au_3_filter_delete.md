---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.899734'
original_url: https://thebuildingcoder.typepad.com/blog/1381_au_3_filter_delete.html
post_number: '1381'
reading_time_minutes: 5
series: filtering
slug: au_3_filter_delete
source_file: 1381_au_3_filter_delete.md
tags:
- csharp
- doors
- elements
- filtering
- geometry
- python
- references
- revit-api
- rooms
- sheets
- views
- walls
- windows
title: Au 3 Filter Delete
word_count: 971
---

### AU, IoC, Banks and Not To Delete While Iterating
Autodesk University is already nearing its end.
It went by so fast!
I attended a bunch of brilliant classes, took notes during Cyrille Fauvel's
[cloud and mobile expert panel](http://the3dwebcoder.typepad.com/blog/2015/12/autodesk-uni-cloud-and-mobile-expert-panel-qa.html),
and successfully presented my own two,
the [SD10181 – Revit API expert panel](http://thebuildingcoder.typepad.com/blog/2015/12/au-keynote-and-revit-api-panel.html#9)
and [SD11048 – connecting desktop and cloud](http://thebuildingcoder.typepad.com/blog/2015/11/connecting-desktop-and-cloud-room-editor-update.html).
That led to a completely different topic... here are a couple of them:
- [SpatialElementGeometryCalculator bug fix – do not delete while iterating](#2)
- [SpatialElementGeometryCalculator migration to Revit 2016](#3)
- [IoC, the Internet of Cows](#4)
- [Two nice Iain Banks Quarry quotes](#5)
#### SpatialElementGeometryCalculator Bug Fix – Do Not Delete While Iterating
The Revit API added a check to prevent deletion of database elements during the iteration over the results of a filtered element collector.
Here it crops up again.
Arif Hanif attended the class SD11048 on connecting desktop and cloud, and we were able to take time off together afterwards to analyse and fix the issue with
the [SpatialElementGeometryCalculator](http://thebuildingcoder.typepad.com/blog/2015/03/findinserts-retrieves-all-openings-in-all-wall-types.html) that
he reported in
his [comment](http://thebuildingcoder.typepad.com/blog/2015/03/findinserts-retrieves-all-openings-in-all-wall-types.html#comment-2380592080) last week:
> I am finding an issue with the temp delete. I implemented the code as on GitHub. The problem appears in both Revit 2015 and 2016. I went through the code and the issue is in the temp delete...
![SpatialElementGeometryCalculator error](img/spatial_element_geomatry_calculator_error_ah.jpg)
I was able to reproduce the issue on my system right out of the box:
![SpatialElementGeometryCalculator error](img/spatial_element_geomatry_calculator_error.png)
Read the error message. It tells you exactly what is going wrong:
> The iterator cannot proceed due to changes made to the Element table in Revit's database (typically, this can be the result of an Element deletion) at Autodesk.Revit.DB.FilteredElementIterator.MoveNext...
Here is the snippet of code causing the issue:

```
  var roomCol = new FilteredElementCollector( doc )
    .OfClass( typeof( SpatialElement ) );

  foreach( var e in roomCol )
  {
    var room = e as Room;
    if( room == null ) continue;
    if( room.Location == null ) continue;
```

The error message does not appear until much later, though, when existing the code block encapsulating this snippet.
Fixing this problem is very easy: just extract the element ids from the collector and dispose of it before starting to loop over the results and potentially deleting things, e.g., like this:

```
  var roomIds = new FilteredElementCollector( doc )
    .OfClass( typeof( SpatialElement ) )
    .ToElementIds();

  foreach( var id in roomIds )
  {
    var room = doc.GetElement( id ) as Room;
    if( room == null ) continue;
    if( room.Location == null ) continue;
```

Please take note, Håkon :-)
Accordingly, here is my updated [answer](http://thebuildingcoder.typepad.com/blog/2015/03/findinserts-retrieves-all-openings-in-all-wall-types.html#comment-2391775373) to Arif's problem report:
> Thank you for sitting down together with me at Autodesk University today and exploring this issue further.
> First of all, by 'follow-up article', I actually meant this one on [wall area calculation handling multiple openings in multiple walls in multiple rooms](http://thebuildingcoder.typepad.com/blog/2015/04/gross-and-net-wall-area-calculation-enhancement-and-events.html#6).
> Another interesting take on this topic is using [IFCExportUtils to determine door and window area](http://thebuildingcoder.typepad.com/blog/2015/03/ifcexportutils-methods-determine-door-and-window-area.html).
> Secondly, we succeeded in finding and resolving the issue.
> The fix is provided by the new [SpatialElementGeometryCalculator release 2015.0.0.4](https://github.com/jeremytammik/SpatialElementGeometryCalculator/releases/tag/2015.0.0.4).
> The error was caused by deleting elements while iterating over them using a filtered element collector.
> To fix, simply store the filtered element collector result as a list of element ids, close the collector, then iterate over the ids and optionally delete elements after the collector is closed.
Thank you again, Arif, for raising this issue!
By the way, the preferred method to submit a problem report on a GitHub sample is to raise a GitHub issue directly on the repository itself.
#### SpatialElementGeometryCalculator Migration to Revit 2016
With that issue out of the way, I also went ahead and migrated the add-in to Revit 2016.
It was trivially simple, since all I had to do was replace the Revit API .NET assembly DLL references.
The result is captured in [SpatialElementGeometryCalculator release 2016.0.0.0](https://github.com/jeremytammik/SpatialElementGeometryCalculator/releases/tag/2016.0.0.0).
#### IoC, The Internet of Cows
Zebra made an RFID tracking system for cows – aka the [Internet of Cows](http://www.developer.com/daily_news/building-the-internet-of-cows.html), or IoC – so that farmers know when the cows are giving birth – they tend to find a private place and hide. They also closely track the water consumed by the cows, because they realized that this metric can be used to accurately predict how much milk they’ll yield in the coming weeks, which can be combined with demand to predict market prices.
#### Two Nice Iain Banks Quarry Quotes
One of my favourite authors is [Iain Banks](https://en.wikipedia.org/wiki/Iain_Banks), as well as his SciFi alter ego, Iain M. Banks.
I am just finishing off his posthumous book \*[Quarry](https://en.wikipedia.org/wiki/The_Quarry_(Iain_Banks_novel))\*.
Here are two nice quotes from it that I really like:
- \*...the end of the nineties and beginning of the noughties.\*
- \*The opposite of lesson is moron.\*
![Iain Banks](img/400px-IainBanks2009.jpg)
I was unaware of his death until today, writing this post, looking at the Wikipedia entries pointed to above.
Oh dear.