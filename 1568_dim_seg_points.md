---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:16.304268'
original_url: https://thebuildingcoder.typepad.com/blog/1568_dim_seg_points.html
post_number: '1568'
reading_time_minutes: 4
series: geometry
slug: dim_seg_points
source_file: 1568_dim_seg_points.md
tags:
- elements
- filtering
- geometry
- references
- revit-api
- selection
- sheets
- transactions
- views
- walls
title: Dim Seg Points
word_count: 767
---

### Determining Dimension Segment Endoints
A rather hard struggle led to a rather simple solution for determining the start and end points of dimension segments.
In summary, the solution looks like this:
- A `Dimension` element is either single- or multi-segment; these two cases need to be handled separately.
- In case of a single segment, the dimension element itself has a line, an origin and a value; the line is indeed the dimension line. However, it may be unbounded, so you cannot determine the start or end point from it. It does provide a direction, though, and the origin is in the middle, so you can determine the start and end points by going forward and backward along the line by half the length:

```
  direction = line.direction.normal;
  start = origin - half * length * direction;
  end = origin + half * length * direction;
```

- In the case of multiple segments, use an analogue approach on each individual segment.
I implemented and added a new external
command [CmdGetDimensionPoints](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdGetDimensionPoints.cs)
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
I added it
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2018.0.134.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.134.0) that
calculates the segment endpoints accordingly.
Here is a dimension between three parallel walls:
![Dimensioning of three walls](img/dim_three_walls_final.png)
`CmdGetDimensionPoints` adds the green model line markers denoting the dimension origin in the middle of the first segment and at the three start and end points of the two segments calculated along the dimension line from that starting point:
![Dimension origin and segment points](img/dim_three_walls_markers_final.png)
For the full research and discussion leading up to this, please refer to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to retrieve a dimension's segment geometry](https://forums.autodesk.com/t5/revit-api-forum/how-to-retrieve-a-dimension-s-segment-geometry/m-p/7145688).
Many thanks to Fair59 for his important contributions and to Jonathan 'Maisoui' for raising the issue.
Here is the full final implementation based on this:
```csharp
///
/// Return dimension origin, i.e., the midpoint
/// of the dimension or of its first segment.
/// summary>
XYZ GetDimensionStartPoint(
Dimension dim )
{
XYZ p = null;
try
{
p = dim.Origin;
}
catch( Autodesk.Revit.Exceptions.ApplicationException ex )
{
Debug.Assert( ex.Message.Equals( "Cannot access this method if this dimension has more than one segment." ) );
foreach( DimensionSegment seg in dim.Segments )
{
p = seg.Origin;
break;
}
}
return p;
}
///
/// Retrieve the start and end points of
/// each dimension segment, based on the
/// dimension origin determined above.
/// summary>
List GetDimensionPoints(
Dimension dim,
XYZ pStart )
{
Line dimLine = dim.Curve as Line;
if( dimLine == null ) return null;
List pts = new List();
dimLine.MakeBound( 0, 1 );
XYZ pt1 = dimLine.GetEndPoint( 0 );
XYZ pt2 = dimLine.GetEndPoint( 1 );
XYZ direction = pt2.Subtract( pt1 ).Normalize();
if( 0 == dim.Segments.Size )
{
XYZ v = 0.5 \* (double)dim.Value \* direction;
pts.Add( pStart - v );
pts.Add( pStart + v );
}
else
{
XYZ p = pStart;
foreach( DimensionSegment seg in dim.Segments )
{
XYZ v = (double) seg.Value \* direction;
if( 0 == pts.Count)
{
pts.Add( p = ( pStart - 0.5 \* v ) );
}
pts.Add( p = p.Add( v ) );
}
}
return pts;
}
///
/// Graphical debugging helper using model lines
/// to draw an X at the given position.
/// summary>
void DrawMarker(
XYZ p,
double size,
SketchPlane sketchPlane )
{
size \*= 0.5;
XYZ v = new XYZ( size, size, 0 );
Document doc = sketchPlane.Document;
doc.Create.NewModelCurve( Line.CreateBound(
p - v, p + v ), sketchPlane );
v = new XYZ( size, -size, 0 );
doc.Create.NewModelCurve( Line.CreateBound(
p - v, p + v ), sketchPlane );
}
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiapp = commandData.Application;
UIDocument uidoc = uiapp.ActiveUIDocument;
Document doc = uidoc.Document;
Selection sel = uidoc.Selection;
ISelectionFilter f
= new JtElementsOfClassSelectionFilter();
Reference elemRef = sel.PickObject(
ObjectType.Element, f, "Pick a dimension" );
Dimension dim = doc.GetElement( elemRef ) as Dimension;
XYZ p = GetDimensionStartPoint( dim );
List pts = GetDimensionPoints( dim, p );
int n = pts.Count;
Debug.Print( "Dimension origin at {0} followed "
+ "by {1} further point{2}{3} {4}",
Util.PointString( p ), n,
Util.PluralSuffix( n ), Util.DotOrColon( n ),
string.Join( ", ", pts.Select(
q => Util.PointString( q ) ) ) );
List d = new List( n );
XYZ q0 = p;
foreach( XYZ q in pts )
{
d.Add( q.X - q0.X );
q0 = q;
}
Debug.Print(
"Horizontal distances in metres: "
+ string.Join( ", ", d.Select( x =>
Util.RealString( Util.FootToMetre( x ) ) ) ) );
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Draw Point Markers" );
SketchPlane sketchPlane = dim.View.SketchPlane;
double size = 0.3;
DrawMarker( p, size, sketchPlane );
pts.ForEach( q => DrawMarker( q, size, sketchPlane ) );
tx.Commit();
}
return Result.Succeeded;
}
```