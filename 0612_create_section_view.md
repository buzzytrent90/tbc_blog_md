---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: code_example
optimization_date: '2025-12-11T11:44:14.264083'
original_url: https://thebuildingcoder.typepad.com/blog/0612_create_section_view.html
post_number: '0612'
reading_time_minutes: 4
series: views
slug: create_section_view
source_file: 0612_create_section_view.htm
tags:
- csharp
- elements
- family
- geometry
- revit-api
- selection
- views
- walls
title: Section View Creation
word_count: 780
---

### Section View Creation

I mentioned some basics of
[elevation and section view creation](http://thebuildingcoder.typepad.com/blog/2009/09/elevation-view.html)
a long time ago, but omitted to publicise the results of the ensuing discussion
[with](http://thebuildingcoder.typepad.com/blog/2009/09/elevation-view.html?cid=6a00e553e1689788330133f4c6fea6970b#comment-6a00e553e1689788330133f4c6fea6970b)
[Konstanty](http://thebuildingcoder.typepad.com/blog/2009/09/elevation-view.html?cid=6a00e553e168978833013487e73089970c#comment-6a00e553e168978833013487e73089970c)
and similar ones
[with
[Jenney](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-university-2010-class-materials.html?cid=6a00e553e168978833013489161808970c#comment-6a00e553e168978833013489161808970c)
and
[Bhavana](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html?cid=6a00e553e168978833014e600d7423970c#comment-6a00e553e168978833014e600d7423970c).
The issue came up in some ADN cases as well, so it really is time to get a bit more information on this out there to you.

**Question:** I see that the CreateViewSection SDK creates a detail view of a wall seen from one side.
How can I create similar top and front views of it?

**Answer:** First of all, as always, of course, take a look at the developer guide, e.g. section 18.1.4 Creating and Deleting Views.
It explains the different view types and how they are defined and created programmatically.

The main aspect in defining the section view direction is setting up the BoundingBoxXYZ appropriately.
As you say, this is demonstrated by the CreateViewSection SDK sample.
It sets up the bounding box used in the call to the NewViewSection method in a helper method GenerateBoundingBoxXYZ.
The latter creates a new BoundingBoxXYZ and sets its Min and Max properties.
The most important thing is its Transform property, which defines the origin and directions (RightDirection, UpDirection and ViewDirection) of the created view.
That is defined by the GenerateTransform helper method, which branches off into different flavours depending on the selected element type, wall, beam or floor.
In the case of a wall, it uses the wall location line to determine its midpoint and direction.

Here is some simpler code that does the same thing without branching out into any helper functions, uses the element bounding box to determine the view origin, and a hard-wired view direction to set up a front view:
```vbnet
  Dim uiapp As UIApplication = commandData.Application
  Dim uidoc As UIDocument = uiapp.ActiveUIDocument
  Dim app As Application = uiapp.Application
  Dim doc As Document = uidoc.Document

  Dim sel As Selection = uidoc.Selection

  Dim e As Element = sel.Elements(0)

  ' get the element bounding box;
  ' 'nothing' means model geometry:

  Dim ebbox As BoundingBoxXYZ =
    e.BoundingBox(Nothing)

  ' determine size of box

  Dim w As Double = (ebbox.Max.X - ebbox.Min.X)
  Dim d As Double = (ebbox.Max.Y - ebbox.Min.Y)
  Dim h As Double = (ebbox.Max.Z - ebbox.Min.Z)

  ' make it easier to see, e.g., not too narrow

  If w < 10.0 Then
    w = 10.0
  End If
  If d < 10.0 Then
    d = 10.0
  End If

  ' from front

  Dim maxPt As XYZ = New XYZ(w, h, 0.0)
  Dim minPt As XYZ = New XYZ(-w, -h, -d)

  Dim bbox As BoundingBoxXYZ = New BoundingBoxXYZ()

  bbox.Enabled = True
  bbox.Max = maxPt
  bbox.Min = minPt

  ' set the transform

  Dim trans As Transform = Transform.Identity

  ' find the mid point of the element

  Dim midPt As XYZ = 0.5 \* (ebbox.Max + ebbox.Min)

  ' set it as origin

  trans.Origin = midPt

  ' determine view direction

  trans.BasisX = XYZ.BasisX
  trans.BasisY = XYZ.BasisZ
  trans.BasisZ = -XYZ.BasisY

  bbox.Transform = trans

  ' create the section view

  Dim viewSection As ViewSection =
    doc.Create.NewViewSection(bbox)
```

To set up a different view, e.g. the right-hand side, you just modify the following lines:
```csharp
  ' . . .
  ' from right
  Dim maxPt As XYZ = New XYZ(d, h, 0.0)
  Dim minPt As XYZ = New XYZ(-d, -h, -w)
  ' . . .
  ' determine view direction
  trans.BasisX = XYZ.BasisY
  trans.BasisY = XYZ.BasisZ
  trans.BasisZ = XYZ.BasisX
```

For the sake of completeness and your comfort, here is [BackSectionView.zip](zip/BackSectionView.zip) containing the rough draft of two entire little VB and C# sample projects.
The C# version implements three methods GetFrontView, GetBackView and GetTopView, which may or may not fulfil the promise of their names, but at least they will give you a starting point.

#### Detail View, Not Section View

When using the NewViewSection method, you should be aware of the note at the end of the developer guide section 18.5 ViewSection, which points out that these views will appear as details views in the project browser, unlike section views created manually using section using the command View > New > Section, which show up under the Sections (Building Section) node in the Project Browser.](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-university-2010-class-materials.html?cid=6a00e553e1689788330134890c7687970c#comment-6a00e553e1689788330134890c7687970c)