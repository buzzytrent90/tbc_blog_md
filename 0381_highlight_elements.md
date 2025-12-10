---
post_number: "0381"
title: "Highlight Elements"
slug: "highlight_elements"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'schedules', 'selection', 'walls']
source_file: "0381_highlight_elements.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0381_highlight_elements.html"
---

### Highlight Elements

I arrived in Boston and am now acclimatising in preparation for the
[AEC DevCamp](http://thebuildingcoder.typepad.com/blog/2010/03/devcamp-devlabs-and-updated-api-training-schedule.html) conference
([web site](http://devcamps2010.autodeskevents.net/index.cfm?event=cms.page&id=SID3FA5EEFA001D080A3456040FDD1BD93C)) starting
on Monday and the
[DevLabs](http://thebuildingcoder.typepad.com/blog/2010/03/devcamp-devlabs-and-updated-api-training-schedule.html#devlabs) in Waltham following afterwards
([register](http://usa.autodesk.com/adsk/servlet/item?id=6703509&siteID=123112&cname=DevLab (AEC), Waltham, Jun 10 2010, 201039)).

Meanwhile, here is a quick
[question from Soub](http://thebuildingcoder.typepad.com/blog/2010/05/pre-post-and-pick-select.html?cid=6a00e553e1689788330133efe49023970b#comment-6a00e553e1689788330133efe49023970b) that
we have actually provided a solution for a couple of times in the past but not explicitly pointed out yet.

**Question:** I have a question related to the API for highlighting elements in Revit Architecture based on the Element ID input.
Could you please help me by explaining how to highlight multiple elements based on IDs?

**Answer:** There is really no difference between highlighting a single element or multiple ones.
All you need to do is add them to the current selection set managed by the Revit UI document, or to the third argument to the Execute method, and then return a Cancelled or Failed result code.

An example of the first approach is provided by the command
[highlighting south facing walls](http://thebuildingcoder.typepad.com/blog/2010/01/south-facing-walls.html).
The main lines of interest there in this context are the ones accessing the current selection set, adding specific elements to it, and setting the property to the new collection:
```csharp
SelElementSet selElements = Document.Selection.Elements;

IEnumerable<Wall> walls = CollectExteriorWalls();

foreach( Wall wall in walls )
{
  XYZ exteriorDirection = GetExteriorWallDirection( wall );

  bool isSouthFacing = IsSouthFacing( exteriorDirection );

  if( isSouthFacing )
  {
    selElements.Add( wall );
  }
}

Document.Selection.Elements = selElements;
```

The code presented there is based on Revit 2010.
It has been updated for the Revit 2011 API in the DirectionCalculation SDK sample and now looks like this:
```csharp
  UIDocument uiDoc = new UIDocument( Document );

  Autodesk.Revit.UI.Selection.SelElementSet
    selElements = uiDoc.Selection.Elements;

  IEnumerable<Wall> walls
    = CollectExteriorWalls();

  foreach( Wall wall in walls )
  {
    XYZ exteriorDirection
      = GetExteriorWallDirection( wall );

    if( useProjectLocationNorth )
    {
      exteriorDirection
        = TransformByProjectLocation(
          exteriorDirection );
    }

    bool isSouthFacing
      = IsSouthFacing( exteriorDirection );

    if( isSouthFacing )
      selElements.Insert( wall );
  }

  // Select all walls which had the proper direction.
  uiDoc.Selection.Elements = selElements;
```

As said, you can also highlight elements graphically by adding them to the third argument of an external command Execute method and then returning a Cancelled or Failed result code:
```csharp
public Result Execute(
  ExternalCommandData revit,
  ref string message,
  ElementSet elements )
{
  // . . .

  Element e;

  // . . .

  elements.Insert( e );
  return Result.Failed;
}
```

This is demonstrated by the Lab1\_2\_CommandArguments of the
[Revit API introduction labs](http://thebuildingcoder.typepad.com/blog/2010/05/pre-post-and-pick-select.html).