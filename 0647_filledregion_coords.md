---
post_number: "0647"
title: "FilledRegion Coordinates"
slug: "filledregion_coords"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api']
source_file: "0647_filledregion_coords.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0647_filledregion_coords.html"
---

### FilledRegion Coordinates

Here is a
[question](http://thebuildingcoder.typepad.com/blog/2010/11/access-to-sketch-and-sketch-plane.html?cid=6a00e553e1689788330153915d8721970b#comment-6a00e553e1689788330153915d8721970b)
raised by Trang on how to access the coordinates of a filled region.

It demonstrates that you can make (completely unsupported and at your own risk) use of the sequential nature of the element id allocation to determine relationships between Revit database elements and their data that is not otherwise accessible through the Revit API.

**Question:** I try to get coordinates of corners of a FilledRegion but I couldn't.
Could you show me how I can get the boundary of a FilledRegion?

**Answer:** The current Revit API does not provide any direct access to this data.

Meanwhile, however, just creating a filled region on the fly and using RevitLookup to analyse the results, I see that a FilledRegion element was created with the element id 161178.
Immediately before, a sketch with id 161176 was created. Its profile defines a curve array, and the lines are accessible.
Maybe this kind of approach can help you retrieve the data you are looking for?

**Response:** What I want is coordinates of a FilledRegion, so here is my code.
```csharp
public List getBoundaryCorner(
  FilledRegion region,
  Document doc )
{
  List result = new List();

  ElementId id = new ElementId(
    region.Id.IntegerValue - 1 );

  Sketch boundary = doc.get\_Element( id ) as Sketch;

  if( null != boundary )
  {
    CurveArray curBoundary
      = boundary.Profile.get\_Item( 0 );

    if( null != curBoundary )
    {
      foreach( Curve cur in curBoundary )
      {
        XYZ corner = cur.get\_EndPoint( 0 );
        result.Add( corner );
      }
    }
  }
  return result;
}
```

**Answer:** Congratulations on getting it working!

I created a new external command CmdFilledRegionCoords from your solution for The Building Coder samples.
For that, I updated your getBoundaryCorner implementation very slightly like this:
```csharp
List<XYZ> GetBoundaryCorners( FilledRegion region )
{
  List<XYZ> result = new List<XYZ>();

  ElementId id = new ElementId(
    region.Id.IntegerValue - 1 );

  Sketch sketch = region.Document.get\_Element(
    id ) as Sketch;

  if( null != sketch )
  {
    CurveArray curves = sketch.Profile.get\_Item( 0 );

    if( null != curves )
    {
      foreach( Curve cur in curves )
      {
        XYZ corner = cur.get\_EndPoint( 0 );
        result.Add( corner );
      }
    }
  }
  return result;
}
```

This helper method is driven by the following mainline in the Execute method implementation:
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

  List<Element> filledRegions
    = new List<Element>();

  if( Util.GetSelectedElementsOrAll(
    filledRegions, uidoc, typeof( FilledRegion ) ) )
  {
    int n = filledRegions.Count;

    string[] results = new string [n];

    int i = 0;

    foreach( FilledRegion region in
      filledRegions.Cast<FilledRegion>() )
    {
      string desc = Util.ElementDescription( region );

      List<XYZ> corners = GetBoundaryCorners(
        region );

      string result = ( null == corners ) ? "failed"
        : string.Join( ", ",
          corners.ConvertAll<string>(
            p => Util.PointString( p ) )
              .ToArray() );

      results[i++] = string.Format( "{0}: {1}",
        desc, result );
    }
    string s = string.Format(
      "Retrieving corners for {0} filled region{1}{2}",
      n, Util.PluralSuffix( n ), Util.DotOrColon( n ) );

    string t = string.Join( "\r\n", results );

    Util.InfoMsg( s, t );
  }
  return Result.Succeeded;
}
```

Here is a sample filled region to test this code on:
![FilledRegion](img/filledregion.png)

Here are the resulting coordinates as displayed by the new overload of the InfoMsg method using a task dialogue:
![FilledRegion coordinates](img/filledregion_coords.png)

For completeness' sake, here they are in text format as well:

```
Retrieving corners for 1 filled region:

Detail Items <161178 Detail Filled Region>:
  (-19.01,40.84,0),
  (1.64,11.35,0),
  (-35.7,17.93,0),
  (-38.81,21.04,0)
```

Thanks to Trang for raising this topic and testing out the idea.

Here is
[version 2012.0.92.0](zip/bc_12_92.zip) of
The Building Coder samples including the new command CmdFilledRegionCoords.

Some additional notes: the element ids I originally saw in RevitLookup were not consecutive, but two apart.
Trang used consecutive ones, I am assuming after a more detailed analysis than my quick glance.

Obviously, the method described here is completely unsupported and will almost certainly fail in some future version.
You use it completely at your own risk.

Probably, at some point in the future, the Revit API will provide official access to the filled region coordinates, and this workaround will no longer be needed.

Still, it was pretty satisfying and good fun to make use of this undocumented feature, and also hear from Rudolf in his comment below that he is aware of it as well, for a similar use with revision clouds.