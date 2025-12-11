---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.308075'
original_url: https://thebuildingcoder.typepad.com/blog/0067_viewports_and_sheets.html
post_number: '0067'
reading_time_minutes: 2
series: views
slug: viewports_and_sheets
source_file: 0067_viewports_and_sheets.htm
tags:
- csharp
- doors
- elements
- levels
- revit-api
- sheets
- views
- walls
- windows
title: Viewports and Sheets
word_count: 368
---

### Viewports and Sheets

Jochen Reichert of the University of Stuttgart asks: How can I determine all the view ports of a drawing sheet and vice versa?

To determine the view ports of a drawing sheet, you can simply use the ViewSheet Views property.
It returns the set of all views on the sheet.
For the inverse relationship, you can create a mapping of your own as described in the discussion on
[inverting the relationship](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html)
between hosted und hosting elements such as doors, windows and walls.
Here is some code demonstrating this for the viewport and view sheet relationships.
It sets up two mappings, one from each view sheet to all its contained viewports, the other from each viewport to the view sheet containing it, which we expect to be unique:

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

List<Element> sheets = new List<Element>();

doc.get\_Elements( typeof( ViewSheet ), sheets );

// map with key = sheet element id and
// value = list of viewport element ids:

Dictionary<ElementId, List<ElementId>>
  mapSheetToViewport =
  new Dictionary<ElementId, List<ElementId>>();

// map with key = viewport element id and
// value = sheet element id:

Dictionary<ElementId, ElementId>
  mapViewportToSheet =
  new Dictionary<ElementId, ElementId>();

foreach( ViewSheet sheet in sheets )
{
  int n = sheet.Views.Size;

  Debug.WriteLine( string.Format(
    "Sheet {0} contains {1} view{2}: ",
    Util.ElementDescription( sheet ),
    n, Util.PluralSuffix( n ) ) );

  ElementId idSheet = sheet.Id;

  foreach( View v in sheet.Views )
  {
    Debug.WriteLine( "  Viewport "
      + Util.ElementDescription( v ) );

    if( !mapSheetToViewport.ContainsKey( idSheet ) )
    {
      mapSheetToViewport.Add( idSheet,
        new List<ElementId>() );
    }
    mapSheetToViewport[idSheet].Add( v.Id );

    Debug.Assert(
      !mapViewportToSheet.ContainsKey( v.Id ),
      "expected viewport to be caontained"
      + " in only one single sheet" );

    mapViewportToSheet.Add( v.Id, idSheet );
  }
}
```

Here is the output generated for a simple model with three sheets:

```
Sheet Drawing Sheets <127148 Unnamed> contains 3 views:
  Viewport Views <29193 East>
  Viewport Views <29233 South>
  Viewport Views <29273 Site>
Sheet Drawing Sheets <127162 Unnamed> contains 1 view:
  Viewport Views <13073 Level 1>
Sheet Drawing Sheets <127176 Unnamed> contains 1 view:
  Viewport Views <15915 Level 2>
```

Here is
[version 1.0.0.18](http://thebuildingcoder.typepad.com/blog/files/bc10018.zip)
of the complete Visual Studio solution with the new command CmdListViews discussed here.