---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.711185'
original_url: https://thebuildingcoder.typepad.com/blog/1298_door_window_area.html
post_number: '1298'
reading_time_minutes: 8
series: general
slug: door_window_area
source_file: 1298_door_window_area.htm
tags:
- csharp
- doors
- elements
- family
- geometry
- parameters
- revit-api
- rooms
- selection
- sheets
- views
- walls
- windows
title: IFCExportUtils Methods Determine Door and Window Area
word_count: 1552
---

### IFCExportUtils Methods Determine Door and Window Area

Last week we discussed the
[SpatialElementGeometryCalculator sample](https://github.com/jeremytammik/SpatialElementGeometryCalculator) for
[calculating gross and net wall areas](http://thebuildingcoder.typepad.com/blog/2015/03/calculating-gross-and-net-wall-areas.html) that
in turn led to the discovery of the
[FindInserts method to retrieve all openings in a wall](http://thebuildingcoder.typepad.com/blog/2015/03/findinserts-retrieves-all-openings-in-all-wall-types.html).

Both of these were prompted by the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread on
[door/window areas](http://forums.autodesk.com/t5/revit-api/door-window-areas/td-p/5535565) raised
by Phillip Miller of Kiwi Codes Solutions Ltd.

Another question that arose in that thread regards the meaning of the built-in parameter HOST\_AREA\_COMPUTED and how to best calculate the area of a door or window family instance.

By the way, here are some previous discussions on HOST\_AREA\_COMPUTED regarding walls:

- [Selecting all Walls](http://thebuildingcoder.typepad.com/blog/2008/09/selecting-all-w.html)
- [Room and Wall Adjacency](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacency.html)
- [Compound Wall Layer Volumes](http://thebuildingcoder.typepad.com/blog/2009/02/compound-wall-layer-volumes.html)

In the thread on
[door/window areas](http://forums.autodesk.com/t5/revit-api/door-window-areas/td-p/5535565),
Phillip wonders about its meaning applied to family instances:

**Question:** I need to determine the cut areas of windows and doors in a wall. I thought this would be a simple matter of grabbing the BIP HOST\_AREA\_COMPUTED as with my testing that was returning the correct area. It turns out that this is not the case from further testing.

Please refer to the attached RVT file with two doors inserted into a wall. They look similar but one is reporting through RevitLookup 4m2 and the other 2m2.

Why is this? What actually is HOST\_AREA\_COMPUTED reporting?
What is the most reliable way of getting the cut area of windows and doors?

**Answer:** We initially avoided looking at the family instance areas directly – by analysing the wall with and without its openings instead – leaving two other interesting topics open for further exploration:

- The meaning of the HOST\_AREA\_COMPUTED parameter value.
- The use of the ExporterIFCUtils GetInstanceCutoutFromWall and ComputeAreaOfCurveLoops methods.

Peter
([pjohan13](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/2663907))
jumped in and answered:

Even though you solved the problem (congratulations on that by the way) I might be able to provide some additional info on the subject.

During the last couple of months I've been working with something related. In my case the specific surface area of each window/door is the main interest and I have thus been digging into this topic quite a bit.

My findings suggest that the value of HOST\_AREA\_COMPUTED in regards to family instances is, what I can best explain as a sum of the geometric objects 'overlapping' the wall geometry in all 3 axes.

Terrible as that explanation might be, consider the following:

A simple 1000 x 1000 mm window family containing only a sheet of glass with a thickness of 100 mm will report an area of 1.2 square meters, equivalent to what could be measured in 2D plan, section and elevation views. If the geometry is copied in the family so that it contains two sweets of glass, HOST\_AREA\_COMPUTED will report the area of both, adding up to 2.4 square meters. This assumption can be supported by snooping a window hosted by a curtain wall. As the curtain wall has no actual depth/thickness, only one overlapping area exist, e.g. as defined by points the X and Z axes.

Whether I'm right or not doesn't really affect anything, but I thought it was worth sharing anyway   :-)

An easy approach to access the actual surface/opening area that I've found extremely useful resides in the Autodesk.Revit.DB.IFC namespace, more specifically in the ExporterIFCUtils class that has provided other interesting features in the past. The methods GetInstanceCutoutFromWall and ComputeAreaOfCurveLoops are able to handle every oddly shaped window I have ever fed them. And especially the latter is handy in a very wide context   :-)

Unfortunately I have no sample code related to HOST\_AREA\_COMPUTED analysis. My findings on the parameter value are based on manual analysis including family editing, measuring, snooping and comparing through a series of small steps. When I eventually stumbled upon the IFCExportUtils methods, that settled my quest for clarity.

The way I have used them, generally, is pretty straightforward. I attached a very simple sample implementing both methods to display the surface areas of selected windows/doors/curtain panels. I've also attached a sample project containing the earlier mentioned 1000 x 1000 mm single sheet window and a 'zig-zag'-shaped window intended to challenge the method a bit – which it doesn't   :-)

- [SampleExpIfcUtilsWinArea.zip](zip/pj_SampleExpIfcUtilsWinArea.zip)
- [WallAndWindow.rvt](zip/pj_WallAndWindow.rvt)

One beauty of the GetInstanceCutoutFromWall method is that it returns the opening boundary as a curve loop and thus provides additional possibilities related to windows/door. For instance, the perimeter is available through curve lengths or sealant area can be calculated by adding a CurveLoop.CreateViaOffset to the list fed into ComputeAreaOfCurveLoops, etc.

I've tested the methods on windows intersecting the vertical join between two different wall types and windows intersecting the horizontal join in stacked walls, in each case with consistent outcome.

Ideally the GetAreaCutoutFromWall would instead be named \*FromHost to cover skylights as well. But with solid performance like this I can't really ask for more, except maybe the values exposed directly by the FamilyInstance class or curtain wall's orientation vector updating accordingly when flipped   a different discussion, maybe   :-)

I hope that the sample can be of use and look forward to seeing more on this matter on The Building Coder.

Please note that this sample is a bit lazy. Normally I would not recommend unit conversion until 'the last moment', e.g. before printing values in the task dialog. Approaches like the one illustrated in the sample could obviously lead to some unintentional 'ft \* m' scenarios causing unexpected outcome.

Many thanks to Peter for his research and sample material!

#### Implementation

I cleaned up his sample code for publication in the
[ExporterIfcUtilsWinArea GitHub repository](https://github.com/jeremytammik/ExporterIfcUtilsWinArea).

As always, the Revit database length unit is imperial feet, and we are interested in seeing the resulting area in square metres, so an approximate conversion factor will come in handy:

```csharp
  const double \_square\_feet\_to\_square\_metres
    = 0.09290304;
```

The algorithm implemented by Peter in the GetInstanceSurfaceAreaMetric method goes like this:

- For a family instance hosted by a normal wall, we can make use of the ExporterIFCUtils GetInstanceCutoutFromWall and ComputeAreaOfCurveLoops methods to determine its area quite accurately.
- For a curtain wall, we can use the value provided by the HOST\_AREA\_COMPUTED built-in parameter instead.
- Finally, for an unhosted family instance, we use the product of its family symbol width and height parameters.

Here is the implementation of that:

```csharp
  /// <summary>
  /// Return surface area of given family instance
  /// in square metres.
  /// </summary>
  static double GetInstanceSurfaceAreaMetric(
    FamilyInstance familyInstance )
  {
    double area\_sq\_ft = 0;

    Wall wall = familyInstance.Host as Wall;

    if( null != wall )
    {
      if( wall.WallType.Kind == WallKind.Curtain )
      {
        area\_sq\_ft = familyInstance.get\_Parameter(
          BuiltInParameter.HOST\_AREA\_COMPUTED )
            .AsDouble();
      }
      else
      {
        Document doc = familyInstance.Document;
        XYZ basisY = XYZ.BasisY;

        // using I = Autodesk.Revit.DB.IFC;

        CurveLoop curveLoop = I.ExporterIFCUtils
          .GetInstanceCutoutFromWall( doc, wall,
            familyInstance, out basisY );

        IList<CurveLoop> loops
          = new List<CurveLoop>( 1 );

        loops.Add( curveLoop );

        area\_sq\_ft = I.ExporterIFCUtils
          .ComputeAreaOfCurveLoops( loops );
      }
    }
    else
    {
      double width
        = familyInstance.Symbol.get\_Parameter(
          BuiltInParameter.FAMILY\_WIDTH\_PARAM )
            .AsDouble();

      double height
        = familyInstance.Symbol.get\_Parameter(
          BuiltInParameter.FAMILY\_HEIGHT\_PARAM )
            .AsDouble();

      area\_sq\_ft = width \* height;
    }
    return \_square\_feet\_to\_square\_metres \* area\_sq\_ft;
  }
```

The element selection and result reporting is handled like this by the external command mainline Execute method:

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

    Selection sel = uidoc.Selection;

    StringBuilder sb = new StringBuilder();
    double areaTotal = 0;

    IEnumerable<ElementId> elementIds = sel.GetElementIds();

    foreach( ElementId elementId in elementIds )
    {
      FamilyInstance fi = doc.GetElement( elementId )
        as FamilyInstance;

      if( null != fi )
      {
        double areaMetric =
            GetInstanceSurfaceAreaMetric( fi );
        areaTotal += areaMetric;

        double areaRound = Math.Round( areaMetric, 2 );

        sb.AppendLine();
        sb.Append( "ElementId: " + fi.Id.IntegerValue );
        sb.Append( "  Name: " + fi.Name );
        sb.AppendLine( "  Area: " + areaRound + " m2" );
      }
    }
    int count = elementIds.Count<ElementId>();

    double areaPrintFriendly = Math.Round( areaTotal, 2 );

    sb.AppendLine( "\nTotal area: "
      + areaPrintFriendly + " m2" );

    TaskDialog taskDialog = new TaskDialog(
      "Selection Area" );

    taskDialog.MainInstruction = "Elements selected: "
      + count;

    taskDialog.MainContent = sb.ToString();

    taskDialog.Show();

    return Result.Succeeded;
  }
```

#### Sample Model

Peter's sample model contains three different kinds of windows:

![ExporterIFCUtils sample model](img/ExporterIfcUtilsWinArea_model.png)

The sample code calculates the following resulting areas for it:

![ExporterIFCUtils sample result](img/ExporterIfcUtilsWinArea_result.png)

#### Download

The entire source code, Visual Studio solution and add-in manifest is provided in the
[ExporterIfcUtilsWinArea GitHub repository](https://github.com/jeremytammik/ExporterIfcUtilsWinArea),
and the version discussed above is
[release 2015.0.0.1](https://github.com/jeremytammik/ExporterIfcUtilsWinArea/releases/tag/2015.0.0.1).