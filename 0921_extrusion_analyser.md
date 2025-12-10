---
post_number: "0921"
title: "Extrusion Analyser and Plan View Boundaries"
slug: "extrusion_analyser"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'parameters', 'python', 'revit-api', 'rooms', 'schedules', 'views']
source_file: "0921_extrusion_analyser.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0921_extrusion_analyser.html"
---

### Extrusion Analyser and Plan View Boundaries

Continuing the research and development for my
[cloud-based round-trip 2D Revit model editing project](http://thebuildingcoder.typepad.com/blog/2013/03/cloud-mobile-extensible-storage-data-use-in-schedules.html#3),
I looked at using the ExtrusionAnalyzer to create a plan view boundary profile for the furniture and equipment family instances and implemented a utility method
[SortCurvesContiguous](http://thebuildingcoder.typepad.com/blog/2013/03/sort-and-orient-curves-to-form-a-contiguous-loop.html) to
sort and re-orient the curves it returns into a closed contiguous loop.

I did not yet display the code driving the extrusion analyser, though, or discuss my experiences with that.

Those are the topics of today's post:

- [Using the ExtrusionAnalyzer class](#2)
- [Handling ExtrusionAnalyzer failures](#3)
- [Testing the ExporterIFCUtils ValidateCurveLoops method](#4)
- [First conclusion and capitulation](#5)
- [Reimplementation with a Boolean union](#6)
- [Conclusion, next steps and download](#7)
- [New Revit Add-ins](#8)

#### Using the ExtrusionAnalyzer Class

Using the extrusion analyser is simple:

**1.** Retrieve and traverse the element geometry:

```python
/// <summary>
/// Retrieve all plan view boundary loops from
/// all solids of given element.
/// </summary>
JtLoops GetPlanViewBoundaryLoopsMultiple(
  Element e,
  ref int nFailures )
{
  Autodesk.Revit.Creation.Application creapp
    = e.Document.Application.Create;

  JtLoops loops = new JtLoops( 1 );

  Options opt = new Options();

  GeometryElement geo = e.get\_Geometry( opt );

  if( null != geo )
  {
    Document doc = e.Document;

    if( e is FamilyInstance )
    {
      geo = geo.GetTransformed(
        Transform.Identity );
    }

    foreach( GeometryObject obj in geo )
    {
      AddLoops( creapp, loops, obj, ref nFailures );
    }
  }
  return loops;
}
```

**2.** For each solid, instantiate an ExtrusionAnalyzer, retrieve and process its resulting output curves:

```python
/// <summary>
/// Add all plan view boundary loops from
/// given solid to the list of loops.
/// The creation application argument is used to
/// reverse the extrusion analyser output curves
/// in case they are badly oriented.
/// </summary>
/// <returns>Number of loops added</returns>
int AddLoops(
  Autodesk.Revit.Creation.Application creapp,
  JtLoops loops,
  GeometryObject obj )
{
  int nAdded = 0;

  Solid solid = obj as Solid;

  if( null != solid
    && 0 < solid.Faces.Size )
  {
    Plane plane = new Plane( XYZ.BasisX,
      XYZ.BasisY, XYZ.Zero );

    ExtrusionAnalyzer extrusionAnalyzer = null;

    extrusionAnalyzer = ExtrusionAnalyzer.Create(
      solid, plane, XYZ.BasisZ );

    Face face = extrusionAnalyzer
      .GetExtrusionBase();

    foreach( EdgeArray a in face.EdgeLoops )
    {
      int nEdges = a.Size;

      List<Curve> curves
        = new List<Curve>( nEdges );

      XYZ p0 = null; // loop start point
      XYZ p; // edge start point
      XYZ q = null; // edge end point

      foreach( Edge e in a )
      {
        Curve curve = e.AsCurve();

        if( \_debug\_output )
        {
          p = curve.get\_EndPoint( 0 );
          q = curve.get\_EndPoint( 1 );
          Debug.Print( "{0} --> {1}",
            Util.PointString( p ),
            Util.PointString( q ) );
        }

        curves.Add( curve );
      }

      CurveUtils.SortCurvesContiguous(
        creapp, curves, \_debug\_output );

      q = null;

      JtLoop loop = new JtLoop( nEdges );

      foreach( Curve curve in curves )
      {
        // Todo: handle non-linear curve.
        // Especially: if two long lines have a
        // short arc in between them, skip the arc
        // and extend both lines.

        p = curve.get\_EndPoint( 0 );

        loop.Add( new Point2dInt( p ) );

        Debug.Assert( null == q
          || q.IsAlmostEqualTo( p, 1e-05 ),
          string.Format(
            "expected last endpoint to equal current start point, not distance {0}",
            (null == q ? 0 : p.DistanceTo( q ))  ) );

        q = curve.get\_EndPoint( 1 );

        if( \_debug\_output )
        {
          Debug.Print( "{0} --> {1}",
            Util.PointString( p ),
            Util.PointString( q ) );
        }

        if( null == p0 )
        {
          p0 = p; // save loop start point
        }
      }
      Debug.Assert( q.IsAlmostEqualTo( p0, 1e-05 ),
        string.Format(
          "expected last endpoint to equal current start point, not distance {0}",
          p0.DistanceTo( q ) ) );

      loops.Add( loop );

      ++nAdded;
    }
  }
  return nAdded;
}
```

#### Handling ExtrusionAnalyzer Failures

The desk solids are processed perfectly well by the extrusion analyser, but two of the chair solids produce failures.
This is the standard Revit content 'Furniture Chair - Office' returning invalid solids.

Since I don't (yet?) know how to detect beforehand whether a solid will cause a failure or not, the only option I see is to encapsulate the extrusion analyser implementation in an exception handler.
I also added code to report how many failures occur:

```csharp
  ExtrusionAnalyzer extrusionAnalyzer = null;

  try
  {
    extrusionAnalyzer = ExtrusionAnalyzer.Create(
      solid, plane, XYZ.BasisZ );
  }
  catch( Autodesk.Revit.Exceptions
    .InvalidOperationException )
  {
    ++nExtrusionAnalysisFailures;
    return nAdded;
  }

  Face face = extrusionAnalyzer
    .GetExtrusionBase();
```

#### Testing the ExporterIFCUtils ValidateCurveLoops Method

Rudolf Honke added a very valid
[suggestion](http://thebuildingcoder.typepad.com/blog/2013/03/sort-and-orient-curves-to-form-a-contiguous-loop.html#comment-6a00e553e168978833017c384939c3970b) to
my
[SortCurvesContiguous implementation](http://thebuildingcoder.typepad.com/blog/2013/03/sort-and-orient-curves-to-form-a-contiguous-loop.html): *You may have a look at ExporterIFCUtils.ValidateCurveLoops method. RevitApi.chm says:
"Does validity checks on a list of curve loops to ensure that they are all co-planar, closed, and properly oriented."
I have never tested this method, but perhaps it prevents you from inventing the wheel another time...*

I like that suggestion a lot and am interested to find out how useful this method might be.

It takes a list of curve loops and returns a new list of curve loops "properly oriented, if possible. If not, the return contains no loops."

I added code to test this method, passing in to it the output produced by the extrusion analyser:

```csharp
  // Test ValidateCurveLoops

  CurveLoop loopIfc = new CurveLoop();

  foreach( Edge e in a )
  {
    Curve curve = e.AsCurve();

    if( \_debug\_output )
    {
      p = curve.get\_EndPoint( 0 );
      q = curve.get\_EndPoint( 1 );
      Debug.Print( "{0} --> {1}",
        Util.PointString( p ),
        Util.PointString( q ) );
    }

    curves.Add( curve );

    // Throws an exception saying "This curve
    // will make the loop not contiguous.
    // Parameter name: pCurve"

    loopIfc.Append( curve );
  }

  // We never reach this point:

  List<CurveLoop> loopsIfc
    = new List<CurveLoop>( 1 );

  loopsIfc.Add( loopIfc );

  IList<CurveLoop> loopsIfcOut = ExporterIFCUtils
    .ValidateCurveLoops( loopsIfc, XYZ.BasisZ );
```

Unfortunately, the CurveLoop Append method throws an exception saying "This curve will make the loop not contiguous. Parameter name: pCurve".

It obviously expects contiguous curves to be passed in and can therefore not be used to re-orient curves if they are oriented the wrong way.

So much for that suggestion.
You got my hopes up there, Rudolf, but no luck this time.

#### First Conclusion and Capitulation

My first (erroneous) conclusion for the extrusion analyser was simple: the output I receive for a plan view is much too complex for my use.

I was expecting it to return the simplest possible contour to represent the shadow cast by the solid passed in.
The results include lots of extraneous loops that do not contribute to the shadow of the object.

Oops.

Since I am passing in the multiple solids from the desk and the chair to the extrusion analyser one by one, individually, it is obviously returning a plan view boundary outline for each one of the solids, individually, as well.

At first glance I thought that this result is much too complicated for me to handle, since all I want is one single boundary for the whole object.

For a moment, I gave up on the whole idea of using the extrusion analyser and decided to switch to a 2D plan view instead, and ask for the view-specific family instance representation in that view. For the desk, that would simply give me the desired rectangular outline.

Capitulation.

Then I switched on my brain for a second again and realised the obvious fact that individual solids will generate individual outlines.

I decided to give the extrusion analyser another go, unite all the solids into one single one, then pass that in to a single call of the extrusion analyser.

#### Reimplementation with a Boolean Union

'Gesagt, getan', as the Germans say, 'No sooner said than done', 'a word and a blow'.

I united all the desk solids, passed the resulting union in to the extrusion analyser, and it produces a single closed loop.

I united all the chair solids, passed the resulting union in to the extrusion analyser, and it produces one single failure.

Oh no!

The individual chair solids causing a failure when passed in individually also cause a failure when united with the unproblematic ones.

Next idea:

- Extract the family instance solids from the geometry one by one.
- Test each solid by passing it in to the extrusion analyser on its own.
- If it causes a failure, discard it.
- If it does not, unite it with the others.
- Pass in the union of all non-failing solids to the extrusion analyser.
- Retrieve the resulting curves.
- Sort and re-orient the curves to form a contiguous closed loop.
- Convert the closed loop data to my integer-based 2D format.

Here is the final code for achieving this:

```python
/// <summary>
/// Retrieve all plan view boundary loops from
/// all solids of given element united together.
/// </summary>
JtLoops GetPlanViewBoundaryLoops(
  Element e,
  ref int nFailures )
{
  Autodesk.Revit.Creation.Application creapp
    = e.Document.Application.Create;

  JtLoops loops = new JtLoops( 1 );

  Options opt = new Options();

  GeometryElement geo = e.get\_Geometry( opt );

  if( null != geo )
  {
    Document doc = e.Document;

    if( e is FamilyInstance )
    {
      geo = geo.GetTransformed(
        Transform.Identity );
    }

    Solid union = null;

    Plane plane = new Plane( XYZ.BasisX,
      XYZ.BasisY, XYZ.Zero );

    foreach( GeometryObject obj in geo )
    {
      Solid solid = obj as Solid;

      if( null != solid
        && 0 < solid.Faces.Size )
      {
        // Some solids, e.g. in the standard
        // content 'Furniture Chair - Office'
        // cause an extrusion analyser failure,
        // so skip adding those.

        try
        {
          ExtrusionAnalyzer extrusionAnalyzer
            = ExtrusionAnalyzer.Create(
              solid, plane, XYZ.BasisZ );
        }
        catch( Autodesk.Revit.Exceptions
          .InvalidOperationException )
        {
          solid = null;
          ++nFailures;
        }

        if( null != solid )
        {
          if( null == union )
          {
            union = solid;
          }
          else
          {
            union = BooleanOperationsUtils
              .ExecuteBooleanOperation( union, solid,
                BooleanOperationsType.Union );
          }
        }
      }
    }
    AddLoops( creapp, loops, union, ref nFailures );
  }
  return loops;
}
```

This is the current result for my simple sample model (copy the text to see the truncated lines in full):

```
Room Rooms <212639 Room 1> has 2 loops:
  0: (2753,3087), (-4446,3087), (-4446,587), (-746,587), (-746,-1212), (2753,-1212)
  1: (298,-112), (298,587), (1698,587), (1698,-112)
FamilyInstance Furniture Desk <212646 1525 x 762mm> has 1 loop:
  0: (664,2561), (664,1761), (2227,1761), (2227,2561), (2056,2561), (2056,2580), (1954,2580), (1954,2561), (937,2561), (937,2580), (836,2580), (836,2561)
FamilyInstance Furniture Desk <212801 1525 x 762mm> has 1 loop:
  0: (-1200,2561), (-1200,1761), (362,1761), (362,2561), (191,2561), (191,2580), (89,2580), (89,2561), (-927,2561), (-927,2580), (-1028,2580), (-1028,2561)
FamilyInstance Furniture Desk <213000 1525 x 762mm> has 1 loop:
  0: (-4135,2561), (-4135,1761), (-2572,1761), (-2572,2561), (-2743,2561), (-2743,2580), (-2845,2580), (-2845,2561), (-3862,2561), (-3862,2580), (-3963,2580), (-3963,2561)
FamilyInstance Furniture Chair - Office <214027 Office Chair>: 2 extrusion analyser failures
FamilyInstance Furniture Chair - Office <214027 Office Chair> has 1 loop:
  0: (-3581,1142), (-3581,1162), (-3561,1162), (-3561,1502), (-3581,1502), (-3581,1542), (-3561,1542), (-3561,1577), (-3021,1577), (-3021,1542), (-3001,1542), (-3001,1502), (-3021,1502), (-3021,1162), (-3001,1162), (-3001,1142), (-3021,1142), (-3021,1108), (-3561,1108), (-3561,1142)
FamilyInstance Furniture Chair - Office <214138 Office Chair>: 2 extrusion analyser failures
FamilyInstance Furniture Chair - Office <214138 Office Chair> has 1 loop:
  0: (-636,1142), (-636,1162), (-616,1162), (-616,1502), (-636,1502), (-636,1542), (-616,1542), (-616,1577), (-76,1577), (-76,1542), (-56,1542), (-56,1502), (-76,1502), (-76,1162), (-56,1162), (-56,1142), (-76,1142), (-76,1108), (-616,1108), (-616,1142)
FamilyInstance Furniture Chair - Office <214409 Office Chair>: 2 extrusion analyser failures
FamilyInstance Furniture Chair - Office <214409 Office Chair> has 1 loop:
  0: (1263,1142), (1263,1162), (1283,1162), (1283,1502), (1263,1502), (1263,1542), (1283,1542), (1283,1577), (1823,1577), (1823,1542), (1843,1542), (1843,1502), (1823,1502), (1823,1162), (1843,1162), (1843,1142), (1823,1142), (1823,1108), (1283,1108), (1283,1142)
```

I have not really checked the validity of these loops yet.

The chairs have arcs in them, so maybe they have to be as complex as they appear.

The desk should actually be just one single simple rectangle, so maybe there is a possibility to clean up its loop, e.g. reduce the 12 vertices to just four, e.g. by identifying collinear segments or something.

I'll look at that in more detail anon.

As you can see, this is all very experimental work in progress.

I hope you can get some use out of it anyway, and am excited to see where this will lead me.

#### Conclusion, Next Steps and Download

It works so far, and I will still have hope of using this for my final project implementation.

The next step is to test the validity of the loops I retrieve.
As said, the end goal is:

- Upload the boundary loop and other data to the cloud.
- Visualise and manipulate the loops in the
  [room layout editor](http://thebuildingcoder.typepad.com/blog/2013/02/2d-svg-editing-on-mobile-device-with-rapha%C3%ABl.html).

Alternatively, I could first implement a visualisation tool in .NET for local use and testing.
I have had that on my list for a long time anyway.

Anyway, here is
[GetFurnitureLoops.zip](zip/GetFurnitureLoops.zip) containing
the complete source code, Visual Studio solution and add-in manifest of the current state of this external command.

#### New Revit Add-ins

Before closing, here are some new Revit add-ins pointed out by developers that have been active here on the blog:

Israel Rodriguez of [icubY](http://icuby.com.br) released his
[mYbox](http://myboxfree.com) add-in
providing an easy way to integrate families that are on our portal directly inside Revit, AutoCAD and SketchUp, and includes a WPF client using the Revit API to instantiate the families
([video](http://www.youtube.com/channel/UCngdwrPsKGkJgXVOrAlzPig/videos)).
The content is mainly in Brazilian Portuguese.

Fernando Malard of
[ofcdesk](http://ofcdesk.com)
points out his
[ofctools](http://www2.ofcdesk.com/tools),
providing advanced maintenance commands to optimise and streamline work, e.g. creating, editing or deleting large numbers of elements.