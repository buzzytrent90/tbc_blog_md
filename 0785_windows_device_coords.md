---
post_number: "0785"
title: "UIView and Windows Device Coordinates"
slug: "windows_device_coords"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'views', 'windows']
source_file: "0785_windows_device_coords.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0785_windows_device_coords.html"
---

### UIView and Windows Device Coordinates

As I pointed out in the discussion of the new
[Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html) features,
the add-in integration enhancements include a new View API, and it provides a new UIView class.

The UIView class represents a view window, enables you to pan, zoom and determine its size for tiling etc., and may be useful for switchback functionality from other visualization environments, for example.

I am very surprised that I have not heard anything more from the developer community about this class yet, nor received any cases querying it and its behaviour.

Maybe the entire humanity falls into just one of two classes: those who missed it entirely, and those who noticed and made use of it without any problem whatsoever.

Anyway, after waiting for a couple of months now and still hearing nothing about it from anyone else, I decided to try it out myself, and I must say I belong to the second group.

The UIView class provides only three methods of interest to us:

- GetWindowRectangle returning the rectangle containing the coordinates of the view's drawing area, in Windows coordinates.- GetZoomCorners returning the corners of the view's rectangle, in Revit model coordinates.- ZoomAndCenterRectangle, which zooms and centres the view to a specified rectangle.

The reason I find this so utterly exciting is that this is the first time in the history of mankind that we have any official possibility at all to correlate between Windows and Revit coordinates, opening unlimited new interaction possibilities.
For instance, how about displaying your own tooltip during the Idling event?

I created a little external command WinCoords which exercises all three of these methods.

By the way, the only way that I saw to access a UIView instance was using the UIDocument GetOpenUIViews method, which returns a list of all of them.

Since each UIView has a property ViewId, we can easily pick the one corresponding to the document active view.

Here is my command implementation, which

- Gets the active view.- Gets all UIView instances.- Determines the UIView for the active view.- Queries and reports its Windows and Revit coordinates.- Calculates new Revit coordinates to zoom in by 10%.- Calls ZoomAndCenterRectangle to do so.

Short and sweet, and a read-only command, of course:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;
  View view = doc.ActiveView;
  UIView uiview = null;
  IList<UIView> uiviews = uidoc.GetOpenUIViews();

  foreach( UIView uv in uiviews )
  {
    if( uv.ViewId.Equals( view.Id ) )
    {
      uiview = uv;
      break;
    }
  }

  Rectangle rect = uiview.GetWindowRectangle();
  IList<XYZ> corners = uiview.GetZoomCorners();
  XYZ p = corners[0];
  XYZ q = corners[1];

  string msg = string.Format(
    "UIView Windows rectangle: {0}; "
    + "zoom corners: {1}-{2}; "
    + "click Close to zoom in by 10%.",
    RectangleString( rect ),
    PointString( p ), PointString( q ) );

  TaskDialog.Show( "WinCoords", msg );

  // Calculate new zoom corners to
  // zoom in by 10%, i.e. the two corners
  // end up at 0.45 of their previous
  // distance to the centre point.

  XYZ v = q - p;
  XYZ center = p + 0.5 \* v;
  v \*= 0.45;
  p = center - v;
  q = center + v;

  uiview.ZoomAndCenterRectangle( p, q );

  return Result.Succeeded;
}
```

This is the task dialogue reporting the results in the 3D view of the rac\_basic\_sample\_project.rvt:
![WinCoords](img/WinCoords.png)

After clicking 'Close', the view is zoomed in by 10 percent, as expected.

Since Windows coordinates have the Y direction reversed to normal systems, the rectangle display string is generated like this:
```csharp
  /// <summary>
  /// Return a string representing the
  /// given Rectangle instance size, in
  /// the order left+top, right+bottom.
  /// Windows coordinates reverse the Y
  /// direction!
  /// </summary>
  static string RectangleString( Rectangle r )
  {
    return string.Format( "({0},{1})-({2},{3})",
      r.Left, r.Top, r.Right, r.Bottom );
  }
```

Here is my little
[WinCoords sample](zip/WinCoords.zip) including
the entire source code, Visual Studio solution and add-in manifest for this command.

I hope you find this useful and am excited to see what you come up with making use of this.