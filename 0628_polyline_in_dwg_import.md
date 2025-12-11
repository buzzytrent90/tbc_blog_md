---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.292900'
original_url: https://thebuildingcoder.typepad.com/blog/0628_polyline_in_dwg_import.html
post_number: 0628
reading_time_minutes: 3
series: general
slug: polyline_in_dwg_import
source_file: 0628_polyline_in_dwg_import.htm
tags:
- csharp
- elements
- geometry
- references
- revit-api
- selection
- views
title: Polylines in Imported DWG Files
word_count: 614
---

### Polylines in Imported DWG Files

I still keep discovering functionality in Revit 2012 API that I have not yet discussed here, such as the following important enhancement:

Previous versions of the Revit API failed to retrieve polyline data from imported DWG files.
The polylines were represented by an internal Revit class which was not exposed through the API, so the geometry retrieval simply ignored these objects.

Happily, that changed in the Revit 2012 API with the introduction of the PolyLine class.
As stated by the Revit API help file RevitAPI.chm, it represents a polyline in space defined by a set of coordinate points.

The 'What's New' section explains:

#### PolyLine returned from Element.Geometry

A new geometry object called a PolyLine is exposed through the API.
The PolyLine represents a set of coordinate points forming contiguous line segments.
Typically this type of geometry would be seen in geometry imported from other formats (such as DWG).
Previous Element.Geometry[] would skip extraction of these geometry object completely.

Here is some sample code making use of this class, simply counting the number of curves and polylines returned by a DWG import instance.
It makes use of two helper functions.

The first helper method is trivial.
It simply returns an English plural suffix for the given number of items, i.e. 's' for zero or more than one items and nothing for exactly one, for prettier printing of the results:
```csharp
static string PluralSuffix( int n )
{
  return 1 == n ? "" : "s";
}
```

The second one, GetSingleSelectedElement, is a bit more substantial.
It retrieves a single pre-selected element, if one exists, and otherwise prompts the user to select an element, including the
[check for a valid selection view](http://thebuildingcoder.typepad.com/blog/2011/08/pickobject-requires-valid-view.html) that we recently discussed:
```csharp
static Result GetSingleSelectedElement(
  UIDocument uidoc,
  out Element e )
{
  e = null;

  Selection sel = uidoc.Selection;

  // Check for a single pre-selected element

  SelElementSet set = sel.Elements;

  if( 1 == set.Size )
  {
    foreach( Element e2 in set )
    {
      e = e2;
    }
  }

  if( null == e )
  {
    // Prompt user to select an element

    if( ViewType.Internal == uidoc.ActiveView.ViewType )
    {
      TaskDialog.Show( "Error",
        "Cannot pick an element in this view: "
        + uidoc.ActiveView.Name );

      return Result.Failed;
    }

    try
    {
      Reference r = sel.PickObject(
        ObjectType.Element,
        "Please pick an element" );

      e = uidoc.Document.get\_Element(
        r.ElementId );
    }
    catch( OperationCanceledException )
    {
      return Result.Cancelled;
    }
  }
  return Result.Succeeded;
}
```

With these two in place, we can proceed to the actual code to retrieve the element geometry and extract polylines from it, provided it is in fact an import instance containing any such polylines:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  Element e;

  Result r = GetSingleSelectedElement( uidoc, out e );

  if( Result.Succeeded == r )
  {
    int curveCounter = 0;
    int polylineCounter = 0;

    GeometryElement geoElement
      = e.get\_Geometry( new Options() );

    foreach( GeometryObject geoObject
      in geoElement.Objects )
    {
      GeometryInstance instance
        = geoObject as GeometryInstance;

      if( null != instance )
      {
        foreach( GeometryObject instObj
          in instance.SymbolGeometry.Objects )
        {
          if( instObj is Curve )
          {
            ++curveCounter;
          }
          else if( instObj is PolyLine )
          {
            ++polylineCounter;
          }
        }
      }
    }
    TaskDialog.Show( "GeometryInstance Symbol Geometry",
      string.Format( "{0} curve{1} and {2} polyline{3}",
        curveCounter, PluralSuffix( curveCounter ),
        polylineCounter, PluralSuffix( polylineCounter ) ) );
  }
  return r;
}
```

I tested it on the following simple model containing one imported DWG file representing a rectangular column:
![Pinned column import instance](img/import_instance_pinned_column.png)

It reports one single polyline representing the geometry:
![Import instance geometry containing polyline](img/import_instance_geometry_result.png)

Unfortunately, as said, this geometry could not be retrieved in previous versions of the Revit API.

Here is
[ImpInstGeo.zip](zip/ImpInstGeo.zip) containing
the complete Visual Studio solution, C# source code, and add-in manifest file of this command.