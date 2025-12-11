---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.376795'
original_url: https://thebuildingcoder.typepad.com/blog/1138_sheet_viewport_loops.html
post_number: '1138'
reading_time_minutes: 4
series: views
slug: sheet_viewport_loops
source_file: 1138_sheet_viewport_loops.htm
tags:
- csharp
- elements
- family
- geometry
- levels
- parameters
- revit-api
- rooms
- sheets
- views
- windows
title: Determining the Size and Location of Viewports on a Sheet
word_count: 709
---

### Determining the Size and Location of Viewports on a Sheet

Today, let's look at how to determine the size and location of a sheet and the views displayed by it.

This is part of the first and most Revit-related of the three enhancement goals for my simplified 2D BIM editor:

- Display selected BIM elements in their sheet and view context – in addition to pure model space room and family instance elements.
- Display and edit non-graphical parameters – in addition to the purely graphical family instance outlines.
- Implement more graphical editing using translation and rotation grip points – in addition to the simple rotation button.

In previous versions of Revit, determining the size and locations of views on a sheet was a pretty challenging undertaking.

Here are some related explorations:

- [List all sheets and their views – CmdListViews](http://thebuildingcoder.typepad.com/blog/2009/01/viewports-and-sheets.html)
- [Determine sheet size – CmdSheetSize](http://thebuildingcoder.typepad.com/blog/2010/05/determine-sheet-size.html)
- [Determine view location on sheet – using Viewport bounding box](http://thebuildingcoder.typepad.com/blog/2010/09/view-location-on-sheet.html)
- [Exact viewport positioning – using Viewport Outline](http://thebuildingcoder.typepad.com/blog/2013/10/exact-viewport-positioning-conceptual-design-automation-and-graitec.html#2)

A quote from the third:
"The View class has a very important property Outline.
The Outline property will return the Max and Min point of the closest bounding box, which includes all elements in this view.
The value in this Max and Min the result of the legend view's real Max and Min coordinates subdivided by the view scale.
For more information about Outline property, please refer to the Revit SDK developer guide 'Revit 2011 API Developer Guide.pdf'.
Approximately, the outline Max point is mapping the same Max point of the viewport in view sheet."

Nowadays, the Viewport class provides an Outline property that considerably simplifies this task.

Here is a sheet 'A101 - Level 0, 1 and 3D' displaying three views:

![Sheet displaying three views](img/sheet_viewport_loops_1.png)

My GeoSnoop utility dynamically generates this temporary form displaying the outlines of the sheet and the three views it contains:

![GeoSnoop displaying the sheet and vieport outlines](img/sheet_viewport_loops_2.png)

The top level code in the CmdUploadSheets external command implementation retrieving the sheet and viewport loops and displaying them in a temporary modeless form basically just consists of two lines of code, calling the new GetSheetViewportLoops and the existing GeoSnoop.DisplayLoops methods:

```csharp
  foreach( ViewSheet sheet in sheets )
  {
    JtLoops sheetViewportLoops
      = GetSheetViewportLoops( sheet );

    string sheet\_number = sheet.get\_Parameter(
      BuiltInParameter.SHEET\_NUMBER )
        .AsString();

    caption = string.Format(
      "Sheet and Viewport Loops - {0} - {1}",
      sheet\_number, sheet.Name );

    GeoSnoop.DisplayLoops( revit\_window,
      caption, false, sheetViewportLoops );
  }
```

The GeoSnoop.DisplayLoops method is pretty much unchanged from its last incarnation for displaying
[room and furniture loops using symbols](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html),
except that the family instances and symbol geometry arguments now are optional and default to null.

The interesting new code to retrieve the sheet and viewport rectangles is short and sweet, making use of a couple of extensions I added to the existing Point2dInt and JtLoop classes:

```csharp
  /// <summary>
  /// Return polygon loops representing the size
  /// and location of given sheet and the viewports
  /// it contains.
  /// </summary>
  static JtLoops GetSheetViewportLoops(
    ViewSheet sheet )
  {
    Document doc = sheet.Document;

    List<Viewport> viewports = sheet
      .GetAllViewports()
      .Select<ElementId,Viewport>(
        id => doc.GetElement( id ) as Viewport )
      .ToList<Viewport>();

    int n = viewports.Count;

    JtLoops sheetViewportLoops = new JtLoops( n + 1 );

    BoundingBoxUV bb = sheet.Outline; // (0,0), (2.76,1.95)

    JtBoundingBox2dInt ibb = new JtBoundingBox2dInt(); // (0,0),(840,...)

    ibb.ExpandToContain( new Point2dInt( bb.Min ) );
    ibb.ExpandToContain( new Point2dInt( bb.Max ) );

    JtLoop loop = new JtLoop( 4 );

    loop.Add( ibb.Corners );

    sheetViewportLoops.Add( loop );

    foreach( Viewport vp in viewports )
    {
      XYZ center = vp.GetBoxCenter();
      Outline outline = vp.GetBoxOutline();

      ibb.Init();

      ibb.ExpandToContain(
        new Point2dInt( outline.MinimumPoint ) );

      ibb.ExpandToContain(
        new Point2dInt( outline.MaximumPoint ) );

      loop = new JtLoop( 4 );

      loop.Add( ibb.Corners );

      sheetViewportLoops.Add( loop );
    }
    return sheetViewportLoops;
  }
```

The updated RoomEditorApp source code, Visual Studio solution and add-in manifest live in the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp).

The version discussed above is
[release 2015.0.2.8](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2015.0.2.8).