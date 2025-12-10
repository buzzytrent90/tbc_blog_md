---
post_number: "1182"
title: "CreateLinkReference Sample Code"
slug: "createlinkreference"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'references', 'revit-api', 'rooms', 'transactions', 'views', 'walls']
source_file: "1182_createlinkreference.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1182_createlinkreference.html"
---

### CreateLinkReference Sample Code

The
[Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) introduced
a whole bunch of linked RVT document interaction enhancements:

- Identifying links
- Obtaining linked documents
- Link creation
- Link load and unload
- Link path type
- Conversion of geometric references
- Room tag creation from linked rooms
- Picking in links

They all provide important and useful functionality.

Let's take a look at one of them in particular as well as another issue that just cropped up:

- [Conversion of a geometric reference](#2) in a linked RVT model
- [Triggering a dynamic model updater by specific element ids](#3)

#### Conversion of a Geometric Reference in a Linked RVT Model

As stated in
[What's New in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html),
"these methods enable conversion between Reference objects that reference the contents of the link and the host, respectively:

- Reference.LinkedElementId
- Reference.CreateLinkReference
- Reference.CreateReferenceInLink

This allows an application, for example, to look at the geometry in the link, find the needed face, and convert the reference to that face into a reference in the host suitable for use to place a face-based instance.
Also, they enable you to obtain a reference in the host (e.g. from a dimension or family) and convert it to a reference in the link, suitable for use in Element.GetGeometryObjectFromReference."

This functionality raised some questions, though, e.g. in the recent comments on
[selecting a face in a linked file](http://thebuildingcoder.typepad.com/blog/2012/05/selecting-a-face-in-a-linked-file.html) by
[Rolando](http://thebuildingcoder.typepad.com/blog/2012/05/selecting-a-face-in-a-linked-file.html?cid=6a00e553e16897883301a511dae999970c#comment-6a00e553e16897883301a511dae999970c) and
[Paul Marsland](http://thebuildingcoder.typepad.com/blog/2012/05/selecting-a-face-in-a-linked-file.html?cid=6a00e553e16897883301a73dee4fed970d#comment-6a00e553e16897883301a73dee4fed970d):

**Question:** Could you please give some example code about the use of Reference.CreateLinkReference(RvtLink) to place a face based family in a linked model.

**Question:** Like Rolando, I have been toying (unsuccessfully) with CreateLinkReference since it appeared in the API, documentation and examples of how this is implemented are virtually non existent. I would greatly appreciate a code example showing how a face based family can be attached to a face in a linked file.

**Answer:** Here is a sample code snippet from the development team exercising this functionality.
Note that it includes hard-coded element ids from a specific sample model that you obviously need to modify to suit your own context:

```csharp
  public void AddFaceBasedFamilyToLinks( Document doc )
  {
    ElementId alignedLinkId = new ElementId( 125929 );

    // Get symbol

    ElementId symbolId = new ElementId( 126580 );

    FamilySymbol fs = doc.GetElement( symbolId )
      as FamilySymbol;

    // Aligned

    RevitLinkInstance linkInstance = doc.GetElement(
      alignedLinkId ) as RevitLinkInstance;

    Document linkDocument = linkInstance
      .GetLinkDocument();

    FilteredElementCollector wallCollector
      = new FilteredElementCollector( linkDocument );

    wallCollector.OfClass( typeof( Wall ) );

    Wall targetWall = wallCollector.FirstElement()
      as Wall;

    Reference exteriorFaceRef
      = HostObjectUtils.GetSideFaces(
        targetWall, ShellLayerType.Exterior )
          .First<Reference>();

    Reference linkToExteriorFaceRef
      = exteriorFaceRef.CreateLinkReference(
        linkInstance );

    Line wallLine = ( targetWall.Location
      as LocationCurve ).Curve as Line;

    XYZ wallVector = ( wallLine.GetEndPoint( 1 )
      - wallLine.GetEndPoint( 0 ) ).Normalize();

    using( Transaction t = new Transaction( doc ) )
    {
      t.Start( "Add to face" );

      doc.Create.NewFamilyInstance(
        linkToExteriorFaceRef, XYZ.Zero,
        wallVector, fs );

      t.Commit();
    }
  }
```

#### How to Trigger a Dynamic Model Updater by Specific Element Ids

**Question:** Is it possible to create an IUpdater triggered by changes to specific elements?
Or, in other words, is there a way to make an ElementFilter that only passes a predefined list of element ids?

**Answer:** This query is actually answered by a brief glance at the basic IUpdater documentation:

All updaters are activated based on triggers, defined by calling one of the
[overloads of the UpdaterRegistry.AddTrigger method](http://revitapisearch.com/html/018aac7e-0c20-c988-b6ab-f592d61a4772.htm).

They all take an ElementFilter argument specifying what criteria the modified elements need to fulfil in order to trigger the event.

The element filter can be as specific as you like, e.g. be based on a list of element ids, a single element id, etc., as you can see from the list of
[ElementFilter constructors](http://help.autodesk.com/view/RVT/2014/ENU/?guid=GUID-97F6B396-94D0-4BEC-A4CE-206D24C9F56D).

The situation where an updated is triggered by changes to one single specific element defined by its element id is illustrated by the
[DynamicModelUpdate AssociativeSection SDK sample](http://thebuildingcoder.typepad.com/blog/2011/08/associative-section-view-fix.html#2).

Here is an overview of
[other discussions on the Dynamic Model Updater framework](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.31).