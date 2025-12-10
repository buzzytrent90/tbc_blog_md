---
post_number: "0665"
title: "Retrieving Duct and Pipe Endpoints"
slug: "retrieve_duct_endpoint"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'python', 'revit-api', 'transactions', 'views', 'windows']
source_file: "0665_retrieve_duct_endpoint.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0665_retrieve_duct_endpoint.html"
---

### Retrieving Duct and Pipe Endpoints

Here is a very basic question that may still stump a newcomer to the Revit MEP API:

**Question:** I have just started to learn the MEP API and I am struggling with a problem, which I thought would be fairly easy, but after a day and a half I guess not.
I need to iterate through ducts (done that bit), then decide if it's a round duct (done that too).
Then I want to get the plan view coordinates in order to draw a centre line.

What I really need are x1, y1, x2, y2 for each end Duct so I can bisect to find the centre point.
I am guessing I should be looking within the duct geometry, returned by the Element.get\_Geometry method in C#.
I am not sure how to get at the coordinates, though.

I have also looked at the analytical model, but the ducts I have drawn (rect and round) do not seem to contain one, or at least it isn't returned by the Element.GetAnalyticalModel method, which always returns null.

**Answer:** Congratulations on getting started with the Revit MEP API and solving your first few issues. Sorry to hear that this current problem has you stumped.

Getting the start and end points of a duct element is actually very easy.
A duct is derived from the MEPCurve class, which has a Location property, which contains a LocationCurve, which contains a geometrical Curve, which provides this information via the EndPoint property, which is accessed in C# through the get\_EndPoint method, taking an argument 0 to return the start and 1 for the end point.

So you do not even have to retrieve and analyse the duct geometry.

The AnalyticalModel is used by Revit Structure, and trying to access it in Revit MEP will not work at all and simply return null, just as you discovered.

Here is a sample application which retrieves all duct elements and lists their start and end point coordinates to the Visual Studio debug output window:
```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  FilteredElementCollector a
    = new FilteredElementCollector( doc )
      .OfClass( typeof( Duct ) );

  int nDucts = 0;
  int nCurves = 0;

  foreach( Duct d in a )
  {
    ++nDucts;

    LocationCurve lc = d.Location as LocationCurve;

    Debug.Assert( null != lc,
      "expected duct to have valid location curve" );

    if( null != lc )
    {
      ++nCurves;

      Curve c = lc.Curve;

      Debug.Print( "Duct {0} from {1} to {2}",
        d.Id.IntegerValue,
        PointString( c.get\_EndPoint( 0 ) ),
        PointString( c.get\_EndPoint( 1 ) ) );
    }
  }
  Debug.Print(
    "{0} duct{1} analysed, and {2} curve{3} listed.",
    nDucts, PluralSuffix( nDucts ),
    nCurves, PluralSuffix( nCurves ) );

  return Result.Succeeded;
}
```

Since this command does not need to write to the Revit database, its Transaction attribute can be set to read-only.

Here is the result of running this command in the sample project rme\_basic\_sample\_project.rvt provided in the Program/Samples subdirectory of the Revit MEP root installation folder:

```
Duct 392168 from (60.53,121.53,10.32) to (60.53,110.24,10.32)
Duct 392170 from (28.59,119.19,10.32) to (28.59,110.24,10.32)
Duct 392174 from (39.29,113.74,10.32) to (39.29,110.57,10.32)

. . .

Duct 686628 from (-39.04,21.2,16.84) to (-39.04,21.2,49.52)
Duct 712063 from (115.66,-11.57,10.38) to (115.66,-15,10.38)
727 ducts analysed, and 727 curves listed.
```

Just like ducts, pipes are also derived from the MEPCurve class, so the exact same approach applies to them as well.

Here is
[DuctEndPoints.zip](zip/DuctEndPoints.zip) including the complete source code and Visual Studio solution of this command.