---
post_number: "1642"
title: "Rot Bbfilter Outline"
slug: "rot_bbfilter_outline"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'revit-api', 'selection', 'sheets', 'views']
source_file: "1642_rot_bbfilter_outline.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1642_rot_bbfilter_outline.html"
---

### Bounding Box Filter is Always Axis Aligned
This is pretty obvious, once you think about it, and apparently worth pointing out anyway:
The outline defining a bounding box filter is always aligned with the cardinal axes.
This question was clarified in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`BoundingBox` outline and `BoundingBoxIsInsideFilter`](https://forums.autodesk.com/t5/revit-api-forum/boundingbox-outline-and-boundingboxisinsidefilter/m-p/7921336):
- [Rotating `Min` and `Max` distorts the box](#2)
- [Rotate target elements or use a solid filter](#3)
#### Rotating Min and Max Distorts the Box
\*\*Question:\*\* I have a question about a bounding box.
I tried to get its outline and retrieve the elements inside it.
I encountered 3 cases:
- Rotation = 0 deg – My code works fine with a horizontal bounding box:
![Bounding box rotated 0 degrees](img/rotated_bounding_box_1.png)
- Rotation = 10 ~ 20 deg – I can still get the BoundingBox outline, but cannot select all elements inside:
![Bounding box rotated 10 ~ 20 degrees](img/rotated_bounding_box_2.png)
- Rotation > 45 deg – I cannot get the BoundingBox outline. The error message says, "outline is an empty outline":
![Bounding box rotated > 45 degrees](img/rotated_bounding_box_3.png)
Here is the code; the exception is thrown by the `Outline` constructor:
```csharp
View3D curView3d = doc.ActiveView as View3D;
BoundingBoxXYZ box = curView3d.GetSectionBox();
Transform t = box.Transform;
Outline o = new Outline(
t.OfPoint( box.Min ),
t.OfPoint( box.Max ) );
FilteredElementCollector collector
= new FilteredElementCollector( doc );
BoundingBoxIsInsideFilter bbfilter
= new BoundingBoxIsInsideFilter( o );
IList insideList = collector
.WhereElementIsNotElementType()
.WherePasses( bbfilter )
.Where( x => x.GetTypeId().IntegerValue != -1 )
.Where( x => x.IsPhysicalElement() )
.Select( x => x.Id )
.ToList();
uidoc.Selection.SetElementIds( insideList );
```
#### Rotate Target Elements or Use a Solid Filter
\*\*Answer:\*\* As said, this is pretty obvious, once you think about it.
The outline defining a bounding box filter is always aligned with the cardinal axes.
If you simply grab the `Min` and `Max` point of the bounding box and rotate them far enough, they will end up in positions that specify an empty `Outline`.
Hence the exception.
You seem to be expecting a rotated box. Instead, it creates an axis aligned outline, which won't have the same proportions as the original box (the min and max were rotated).
You can solve this problem by doing the opposite, i.e., rotating the elements' outlines and not the BBox's outline (new outline from all rotated corners, so you lose some precision).
We ended up doing that is one case and it worked pretty well.
Another approach that comes to mind:
Instead of using the [`BoundingBoxIsInsideFilter` class](http://www.revitapidocs.com/2018.1/eb8735d7-28fc-379d-9de9-1e02326851f5.htm) with an axis-aligned bounding box,
you could simply create your own `Solid` representing the rotated box and use
an [`ElementIntersectsSolidFilter`](http://www.revitapidocs.com/2018.1/19276b94-fa39-64bb-bfb8-c16967c83485.htm).
Note that the `BoundingBoxIsInsideFilter` is a quick filter, whereas the `ElementIntersectsSolidFilter` is slow,
cf. [Quick, Slow and LINQ Element Filtering](http://thebuildingcoder.typepad.com/blog/2015/12/quick-slow-and-linq-element-filtering.html).