---
post_number: "0055"
title: "3D Polygon Areas"
slug: "3d_polygon_area"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'walls', 'windows']
source_file: "0055_3d_polygon_area.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0055_3d_polygon_area.html"
---

### 3D Polygon Areas

We continue the discussion initiated in the recent post on
[two-dimensional polygon area calculation](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html).
We can use these areas to determine which of the edge loops is the outer loop versus inner loops defining holes.
The polygon with the largest area is the outer loop, all others are inner holes.
This is a continuation of the analysis for determining the profile boundary loop polygons for
[floor slabs](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html)
and
[walls](http://thebuildingcoder.typepad.com/blog/2008/11/wall-elevation-profile.html).
In this instalment, we expand our analysis to include three-dimensional polygons, i.e. planar polygons oriented arbitrarily in 3D space.

To determine the area of a two-dimensional polygon in the previous post, we used a simple formula which basically works by calculating the area of the triangles spanned by an arbitrary point in the plane and any two consecutive polygon vertices, and summing up all of those triangle areas.
The polygons returned for the floor and wall boundary loops are three dimensional.
We have two choices for calculating the area of a three-dimensional planar polygon in space.
One approach is to transform the polygon appropriately onto the XY plane and make use of the formula we already have.
Another approach makes use of the same algorithm adapted for use on the three-dimensional vertices directly.
It calculates each polygon's plane data in space, i.e. its distance from the origin and normal vector.
The polygon area is by-product of these calculations, because it is equal to half of the length of the non-normalized normal vector.
I would like to demonstrate both approaches, so we can compare them and ensure that both return the same results.
We will begin with the implementation of the three-dimensional algorithm, including optimised implementations for the special cases of triangles and four-sided polygons:

```csharp
bool GetPolygonPlane(
  List<XYZ> polygon,
  out XYZ normal,
  out double dist,
  out double area )
{
  normal = XYZ.Zero;
  dist = area = 0.0;
  int n = ( null == polygon ) ? 0 : polygon.Count;
  bool rc = ( 2 < n );
  if( 3 == n )
  {
    XYZ a = polygon[0];
    XYZ b = polygon[1];
    XYZ c = polygon[2];
    XYZ v = b - a;
    normal = v.Cross( c - a );
    dist = normal.Dot( a );
  }
  else if( 4 == n )
  {
    XYZ a = polygon[0];
    XYZ b = polygon[1];
    XYZ c = polygon[2];
    XYZ d = polygon[3];

    normal.X = ( c.Y - a.Y ) \* ( d.Z - b.Z )
      + ( c.Z - a.Z ) \* ( b.Y - d.Y );
    normal.Y = ( c.Z - a.Z ) \* ( d.X - b.X )
      + ( c.X - a.X ) \* ( b.Z - d.Z );
    normal.Z = ( c.X - a.X ) \* ( d.Y - b.Y )
      + ( c.Y - a.Y ) \* ( b.X - d.X );

    dist = 0.25 \*
      ( normal.X \* ( a.X + b.X + c.X + d.X )
      + normal.Y \* ( a.Y + b.Y + c.Y + d.Y )
      + normal.Z \* ( a.Z + b.Z + c.Z + d.Z ) );
  }
  else if( 4 < n )
  {
    XYZ a;
    XYZ b = polygon[n - 2];
    XYZ c = polygon[n - 1];
    XYZ s = XYZ.Zero;

    for( int i = 0; i < n; ++i ) {
      a = b;
      b = c;
      c = polygon[i];

      normal.X += b.Y \* ( c.Z - a.Z );
      normal.Y += b.Z \* ( c.X - a.X );
      normal.Z += b.X \* ( c.Y - a.Y );

      s += c;
    }
    dist = s.Dot( normal ) / n;
  }
  if( rc )
  {
    double length = normal.Length;
    rc = !Util.IsZero( length );
    Debug.Assert( rc );

    if( rc )
    {
      normal /= length;
      dist /= length;
      area = 0.5 \* length;
    }
  }
  return rc;
}
```

We define a variable n for the number of polygon vertices.
We have implemented specific code for triangles, i.e. the case n = 3, and 4-sided polygons.
The polygon area is half of the length of the non-normalized normal vector of the plane.

Here is the main section of the external command making use of this function.
It lists the areas of the various loops of the selected walls, or all walls in the model, if none were explicitly selected, and highlights the largest area.
Just like in the floor area calculation, the code assumes that only one wall is picked.
If more than one is processed, then the determination of the outer loop will only work for the wall with the largest surface area, since all loops of all walls are included in one single list.
We repackaged the functionality for determining the wall profile polygons into a separate utility method GetWallProfilePolygons() in the CmdWallProfile class, so that we can reuse that functionality from the new command as well:

```csharp
List<List<XYZ>> polygons
  = CmdWallProfile.GetWallProfilePolygons(
    app, walls );

int i = 0, n = polygons.Count;
double[] areas = new double[n];
double d, a, maxArea = 0.0;
XYZ normal;
foreach( List<XYZ> polygon in polygons )
{
  GetPolygonPlane( polygon,
    out normal, out d, out a );
  if( Math.Abs( maxArea ) < Math.Abs( a ) )
  {
    maxArea = a;
  }
  areas[i++] = a;
}

Debug.WriteLine( string.Format(
  "{0} boundary loop{1} found.",
  n, Util.PluralSuffix( n ) ) );

for( i = 0; i < n; ++i )
{
  Debug.WriteLine( string.Format(
    "  Loop {0} area is {1} square feet{2}",
    i,
    Util.RealString( areas[i] ),
    ( areas[i].Equals( maxArea )
      ? ", outer loop of largest wall"
      : "" ) ) );
}
```

Here is a small example model with two walls selected and highlighted in red. Their boundary loops are represented by model lines in green, offset outward from the outer wall face by one foot:

![Wall boundary loops](img/wall_boundary_loops.png)

This is the list of the boundary loop areas of the two selected walls.
The first wall has an outer loop and two inner ones for the two windows.
The second wall has only one single simple rectangular outer loop:

```
4 boundary loops found.
  Loop 0 area is 288.98 square feet, outer loop of largest wall
  Loop 1 area is 2.67 square feet
  Loop 2 area is 2.67 square feet
  Loop 3 area is 172.22 square feet
```

There are still some aspects of this topic left that we would like to address in future posts.
As mentioned above, we would like to transform the 3D polygons into the 2D XY plane
so we can use the two-dimensional GetSignedPolygonArea() on those as well, instead of performing the area calculation in 3D.
Then we can compare the 2D and 3D results to ensure that they are equal.
It would also be interesting to compare the relative speed of transforming the 3D situation to 2D and using 2D area calculation versus direct 3D area calculation.

Here is an updated
[version 1.0.0.15](http://thebuildingcoder.typepad.com/blog/files/bc10015.zip)
of the complete Visual Studio solution,
including the two new commands CmdSlabBoundaryArea and CmdWallProfileArea as well as all other commands discussed so far.