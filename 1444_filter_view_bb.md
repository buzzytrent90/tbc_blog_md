---
post_number: "1444"
title: "The Building Coder"
slug: "filter_view_bb"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'revit-api', 'sheets', 'views']
source_file: "1444_filter_view_bb.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1444_filter_view_bb.html"
---

### Filtering for View Specific Elements
While preparing for
the [Forge DevCon](http://forge.autodesk.com/conference) in SF and
the [Athens Forge meetup](http://www.meetup.com/de-DE/I-love-3D-Athens/events/230543759) and
[web server workshop](http://www.meetup.com/de-DE/I-love-3D-Athens/events/230544059)
at [The Cube Athens](http://thecube.gr),
I also happened to hear about the solution to the question raised by Chema in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
on [deleting an area in a drafting view](http://forums.autodesk.com/t5/revit-api/delete-an-area-in-a-drafting-view/td-p/6342882):
\*\*Question:\*\* I am working on a tool that creates schematics. I need to delete some elements (detail items) in a given area of my drafting view:
![Filter detail item](img/filter_detail_item.png)
In the above example I need to remove the lower detail item and keep the other two. I am trying to use the `ElementOwnerViewFilter` combined with the `BoundingBoxIntersectsFilter` as shown below.
```csharp
private void CleardBArea(
Document doc,
ElementId vId,
int dBPos,
double width,
double height )
{
XYZ min = new XYZ( 0, ( height / 4 ) \* dBPos, 0 );
XYZ max = new XYZ( width, ( height / 4 ) \* ( dBPos + 1 ), 0 );
Outline outline = new Outline( min, max );
BoundingBoxIntersectsFilter bbF
= new BoundingBoxIntersectsFilter( outline );
ElementOwnerViewFilter eOVF
= new ElementOwnerViewFilter( vId );
FilteredElementCollector vColl
= new FilteredElementCollector( doc )
.WherePasses( eOVF )
.WherePasses( bbF );
ClearElements( vColl );
}
```
It works fine if I try to delete model lines, but unfortunately, when I try to delete detail items, the filter ignores it.
Could you please recommend any other way to develop it?
I attached a zip file [filter_detail_item.zip](zip/filter_detail_item.zip) with the minimal information to reproduce my problem: a Revit 2015 project file containing a macro that should remove half of the drafting view. It works with all elements excepts the family instances.
Answer: The `BoundingBoxIntersectsFilter` is intended for use with a model geometry bounding box.
A detail item only has a view specific geometry bounding box, so it fails to filter them.
The workaround is to retrieve the view specific bounding box and use `Outline.Intersects` to perform the equivalent check, e.g., like this:
```csharp
var b1 = detailItem.get_BoundingBox( this.ActiveView );
XYZ min = new XYZ();
XYZ max = new XYZ( 1.969, 0.656, 0 );
var outline = new Outline( min, max );
var outlineOfDetailItem = new Outline( b1.Min, b1.Max );
outline.Intersects( outlineOfDetailItem, 0.00001 );
```