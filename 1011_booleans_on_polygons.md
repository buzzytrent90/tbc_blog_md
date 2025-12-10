---
post_number: "1011"
title: "Boolean Operations for 2D Polygons"
slug: "booleans_on_polygons"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'python', 'references', 'revit-api', 'rooms', 'selection', 'transactions', 'walls']
source_file: "1011_booleans_on_polygons.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1011_booleans_on_polygons.html"
---

### Boolean Operations for 2D Polygons

Once again, I have a special titbit for you to start the week.
Enjoy!

Some of the most important utility tools for analysis of BIM and CAD models in general need to determine simple properties of and relationships between 2D points and polygons, e.g. point in polygon determination, area calculation and Boolean operations for overlapping areas, for instance for
[room and wall adjacent area](http://thebuildingcoder.typepad.com/blog/2009/01/room-and-wall-adjacent-area.html) calculations.

Some of these, but not all, are covered by the Revit API, though often only for a specialised context.

Any serious BIM coder will require generic versions of these tools, and the most reliable way to ensure that is as all-too-often to
[DIY](http://en.wikipedia.org/wiki/Do_it_yourself).

The Building Coder did so right from the beginning of its existence, e.g. for

- [Point in polygon containment](http://thebuildingcoder.typepad.com/blog/2010/12/point-in-polygon-containment-algorithm.html)
- [2D polygon areas and outer loop](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html)
- [3D polygon areas](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html)
- [Boolean operations for 2D polygons](http://thebuildingcoder.typepad.com/blog/2009/02/boolean-operations-for-2d-polygons.html)
- [GetPolygon extension methods](http://thebuildingcoder.typepad.com/blog/2010/02/getpolygon-extension-methods.html)

For the point containment check and area calculation, it is easy enough to implement a reliable stand-alone solution for yourself.

Boolean operations on polygons are a bit harder, however, and back in 2009 we took recourse to an external library to address that.

Here is an update and hopefully improvement to the solution provided then, prompted by the following query:

**Question:** I tried to use the
[GpcNET Boolean operation for polygon](http://thebuildingcoder.typepad.com/blog/2009/02/boolean-operations-for-2d-polygons.html) solution
that you provided.

I cannot make it work, even though I updated it to the Revit 2014 version:

![GpcNET exception](img/gpcnet_exception.jpg)

I tried switching between x86 and x64 configuration in Visual Studio, but that did not help.

Is there a fix for this, or an updated GPC DLL?

Or does the Revit API provide any similar tool to perform 2D polygon Boolean operations?

**Answer:** Actually, I think it would be your job to create that kind of stuff for yourself, especially since you have my previous blog post at hand, showing you how easy it is to do, and listing almost exactly which steps to take.

It is definitely an extremely important tool to have, though, so in this specific case, I will be happy to explore the issue and provide an update to the previous example.

The blog post you refer to is rather old, from the year 2009.

Maybe there is something better around now?

I spent a few moments researching this.
I performed a simple search for
[open source boolean operation 2d polygon](http://lmgtfy.com/?q=open+source+boolean+operation+2d+polygon).

That took me to the well-known Wikipedia article on
[Boolean operations on polygons](http://en.wikipedia.org/wiki/Boolean_operations_on_polygons).

In it, Michael Leonov's
[comparison of polygon clippers](http://www.complex-a5.ru/polyboolean/comp.html) is too old by now, from 1998.

Angus Johnson's
[comparison](http://angusj.com/delphi/clipper.php#features) was
updated in 2013 and led me to download the
[Clipper polygon clipping library](http://angusj.com/delphi/clipper.php).

It is written in Delphi, C++, C# and Python and is currently at version 5.1.6, updated on May 23 2013.
It is freeware for both open source and commercial applications, equipped with the Boost Software License.

I downloaded and unzipped clipper\_ver5.1.6.zip, loaded the C# GuiDemo solution into Visual Studio, hit F5 to debug, and had a result within a few minutes of starting to answer your case:

![Clipper GUI demo](img/clipper_gui_demo.png)

Since this is C# code, you can easily integrate it into your Revit add-in.

Now comes the fun part :-)

Hack, hack, hack.

Ok, I completed my sample app.
It implements an external command named RvtClipper that intersects two slabs, which can be floors or ceilings, and creates a new floor representing the result:

![RvtClipper floor intersection](img/clipper_floor_intersection.png)

Let's take a quick look at the code to achieve this:

- [Clipper library project integration](#2).
- [Clipper and Revit point class translation](#3).
- [GetBoundaryLoops](#4) – return horizontal slab boundary loops.
- [Select two slabs in the BIM](#5).
- [Retrieve boundaries, intersect and create a floor](#6).
- [Download](#7).

#### Clipper Library Project Integration

The Clipper GUI demo sample application solution contains two projects: one for the GUI demo itself, and the other to generate the Clipper library from its one and only C# source code module.

I therefore simply copied that entire project into my add-in solution, made my add-in dependent on the Clipper library and added a reference to its .NET assembly:

![Clipper library integration](img/clipper_library_integration.png)
#### Clipper and Revit Point Class Translation

Part of the Clipper speed and reliability presumably stems from the fact that it is integer-based, eliminating all the rounding problems dealing with real numbers.

It uses an IntPoint class with two Int64 X and Y coordinates.

I have often approximated Revit XYZ vertices using integer-based point classes in the past, so this seems pretty appropriate for me.

Once again, I decided to simply convert the Revit XYZ imperial coordinates to millimetres.
That choice is completely arbitrary.
If you prefer higher precision, please be my guest.

Here is the code I use to convert back and forth between the two systems:

```python
  /// <summary>
  /// Consider a Revit length zero
  /// if is smaller than this.
  /// </summary>
  const double \_eps = 1.0e-9;

  /// <summary>
  /// Conversion factor from feet to millimetres.
  /// </summary>
  const double \_feet\_to\_mm = 25.4 \* 12;

  /// <summary>
  /// Conversion a given length value
  /// from feet to millimetres.
  /// </summary>
  static long ConvertFeetToMillimetres( double d )
  {
    if( 0 < d )
    {
      return \_eps > d
        ? 0
        : (long) ( \_feet\_to\_mm \* d + 0.5 );

    }
    else
    {
      return \_eps > -d
        ? 0
        : (long) ( \_feet\_to\_mm \* d - 0.5 );

    }
  }

  /// <summary>
  /// Conversion a given length value
  /// from millimetres to feet.
  /// </summary>
  static double ConvertMillimetresToFeet( long d )
  {
    return d / \_feet\_to\_mm;
  }

  /// <summary>
  /// Return a clipper integer point
  /// from a Revit model space one.
  /// Do so by dropping the Z coordinate
  /// and converting from imperial feet
  /// to millimetres.
  /// </summary>
  IntPoint GetIntPoint( XYZ p )
  {
    return new IntPoint(
      ConvertFeetToMillimetres( p.X ),
      ConvertFeetToMillimetres( p.Y ) );
  }

  /// <summary>
  /// Return a Revit model space point
  /// from a clipper integer one.
  /// Do so by adding a zero Z coordinate
  /// and converting from millimetres to
  /// imperial feet.
  /// </summary>
  XYZ GetXyzPoint( IntPoint p )
  {
    return new XYZ(
      ConvertMillimetresToFeet( p.X ),
      ConvertMillimetresToFeet( p.Y ),
      0.0 );
  }
```
#### GetBoundaryLoops – Return Horizontal Slab Boundary Loops

Ok, so we now know how to convert a Revit point to a Clipper one.

How do we represent an entire boundary loop, and how can that data be retrieved from a BIM slab?

That is achieved in one fell swoop by the GetBoundaryLoops method, returning a collection of Clipper polygons.

A single slab may generate multiple loops, both due to holes in its interior and due to consisting of several separate pieces:

```csharp
/// <summary>
/// Retrieve the boundary loops of the given slab
/// top face, which is assumed to be horizontal.
/// </summary>
Polygons GetBoundaryLoops( CeilingAndFloor slab )
{
  int n;
  Polygons polys = null;
  Document doc = slab.Document;
  Application app = doc.Application;

  Options opt = app.Create.NewGeometryOptions();

  GeometryElement geo = slab.get\_Geometry( opt );

  foreach( GeometryObject obj in geo )
  {
    Solid solid = obj as Solid;
    if( null != solid )
    {
      foreach( Face face in solid.Faces )
      {
        PlanarFace pf = face as PlanarFace;
        if( null != pf
          && pf.Normal.IsAlmostEqualTo( XYZ.BasisZ ) )
        {
          EdgeArrayArray loops = pf.EdgeLoops;

          n = loops.Size;
          polys = new Polygons( n );

          foreach( EdgeArray loop in loops )
          {
            n = loop.Size;
            Polygon poly = new Polygon( n );

            foreach( Edge edge in loop )
            {
              IList<XYZ> pts = edge.Tessellate();

              n = pts.Count;

              foreach( XYZ p in pts )
              {
                poly.Add( GetIntPoint( p ) );
              }
            }
            polys.Add( poly );
          }
        }
      }
    }
  }
  return polys;
}
```

With all of that in place, we have nothing more to do before proceeding with the external command mainline.

As mostly, it consists of two parts: the user interface to select the two slabs to intersect, and the worker code to do the job.

#### Select Two Slabs in the BIM

The selection supports both pre- and post-selection.

You can select a bunch of elements before launching the command.
In that case, the first two slabs encountered in the selection will be picked.

If you pre-select nothing, all floor elements in the entire model will be retrieved and the first two encountered selected.

As you may have gathered, this is more geared towards effective simple testing than production use:

```python
  // Two slabs to intersect.

  CeilingAndFloor[] slab
    = new CeilingAndFloor[2] { null, null };

  // Access current selection

  Selection sel = uidoc.Selection;

  int n = sel.Elements.Size;

  if( 0 < n )
  {
    foreach( Element e in sel.Elements )
    {
      if( null == slab[0] )
      {
        slab[0] = e as Floor;
      }
      else if( null == slab[1] )
      {
        slab[1] = e as Floor;
      }
      else
      {
        break;
      }
    }
    if( null == slab[0] || null == slab[1] )
    {
      message = "Please select two floor or ceiling slabs to intersect.";
      return Result.Failed;
    }
  }
  else
  {
    // Retrieve elements from database

    FilteredElementCollector floors
      = new FilteredElementCollector( doc )
        .WhereElementIsNotElementType()
        .OfCategory( BuiltInCategory.OST\_Floors );

    foreach( Element e in floors )
    {
      if( null == slab[0] )
      {
        slab[0] = e as Floor;
      }
      else if( null == slab[1] )
      {
        slab[1] = e as Floor;
      }
      else
      {
        break;
      }
    }
    if( null == slab[0] || null == slab[1] )
    {
      message = "Unable to find two floors in project to intersect.";
      return Result.Failed;
    }
  }
```
#### Retrieve Boundaries, Intersect and Create a Floor

We have our two slabs.

Let's go ahead and do the job.

The Revit API imposes one unfortunate caveat: the doc.Create.NewFloor
[floor creation method can only handle a single exterior loop](http://thebuildingcoder.typepad.com/blog/2013/07/create-a-floor-with-an-opening-or-complex-boundary.html).

We therefore ignore all but the first boundary loop returned in the intersection result, and don't even care what that actually represents.

See how simple this is:

```csharp
  // Retrieve the two slabs' boundary loops

  Polygons subj = GetBoundaryLoops( slab[0] );
  Polygons clip = GetBoundaryLoops( slab[1] );

  // Calculate the intersection

  Polygons intersection = new Polygons();

  Clipper c = new Clipper();

  c.AddPolygons( subj, PolyType.ptSubject );

  c.AddPolygons( clip, PolyType.ptClip );

  c.Execute( ClipType.ctIntersection, intersection,
    PolyFillType.pftEvenOdd, PolyFillType.pftEvenOdd );

  // Check for a valid intersection

  if( 0 < intersection.Count )
  {
    // Determine new intersection floor boundary

    // We can only handle a single exterior loop
    // here, unfortunately; cf.
    // http://thebuildingcoder.typepad.com/blog/2013/07\
    // /create-a-floor-with-an-opening-or-complex-boundary.html

    CurveArray curves = app.Create.NewCurveArray();

    Polygon poly = intersection[0];

    IntPoint? p0 = null; // first
    IntPoint? p = null; // previous

    foreach( IntPoint q in poly )
    {
      if( null == p0 )
      {
        p0 = q;
      }
      if( null != p )
      {
        curves.Append(
          Line.CreateBound(
            GetXyzPoint( p.Value ),
            GetXyzPoint( q ) ) );
      }
      p = q;
    }

    // Do the dirty deed

    using( Transaction tx = new Transaction( doc ) )
    {
      tx.Start( "Create Intersection Floor" );
      doc.Create.NewFloor( curves, false );
      tx.Commit();
    }
  }
```

I showed you the result above.

Pretty nifty, huh?

Oh yes, please also note my first use of
[nullable values](http://stackoverflow.com/questions/2735638/nullable-integer-in-net).
Very useful indeed :-)

#### Download

One nice aspect of this version is that the C# version of the Clipper library is provided in its entirety by the clipper\_library project included in my add-in Visual Studio solution, so you do not even need to download it separately yourself to make use of this.

You do need to ensure that the Clipper .NET assembly is located where the add-in assembly can find it, though.
I'll leave that up to you, of course.

Here is
[RvtClipper.zip](zip/RvtClipper.zip) containing
the complete add-in and library source code, solution, and add-in manifest.

I wish you lots of fun and success making use of this, and please let us know what experiences you make and what nice uses you put it to.
Thank you.