---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 11.4
content_type: code_example
optimization_date: '2025-12-11T11:44:14.775637'
original_url: https://thebuildingcoder.typepad.com/blog/0874_centroid.html
post_number: 0874
reading_time_minutes: 24
series: general
slug: centroid
source_file: 0874_centroid.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- geometry
- levels
- python
- references
- revit-api
- rooms
- selection
- views
- walls
- windows
title: Solid Centroid and Volume Calculation
word_count: 4733
---

### Solid Centroid and Volume Calculation

I have not posted anything since last Friday, being too caught up in the West European Developer days and travelling.
Today and tomorrow we spend in Gothenburg on the last lap of our journey, after Paris, Milano, Farnborough and Munich.
I had a nap in the taxis to and from the airport yesterday, both in Germany and Sweden, so I was able to burn some midnight oil to share the following exploration with you.

One of the many interesting conversations I had at Autodesk University that caught my special attention dealt with a simple geometrical Revit API question; geometrical questions are among my favourites, and this one is really basic: how to calculate the
[centre of mass](http://en.wikipedia.org/wiki/Center_of_mass) or
[centroid](http://en.wikipedia.org/wiki/Centroid) of a solid.

Once again, that led to the exploration of a number of interesting little sub-topics:

- [Revit API centroid and volume calculation](#1) methods
- [Gap free triangulation for polyhedral approximation](#2)
- [Polyhedron centroid calculation algorithm](#3)
- [Revit implementation and CentroidVolume class](#4)
- [Polyhedron centroid calculation implementation](#5)
- [Support for multiple solids](#6)
- [Mainline with picking twists](#7)
- [Results](#8)
- [Download](#9)

Before getting into that, here is a quick heads-up to point out a new free technology preview to simulate airflow around buildings or other objects in a virtual wind tunnel:

#### Project Falcon Computational Fluid Dynamics CFD Add-in

Autodesk released
[Falcon for Revit](http://labs.autodesk.com/utilities/falcon) to
[Autodesk Labs](http://labs.autodesk.com).
Project Falcon is an outdoor airflow simulation add-in for Revit:

Emile Kfouri describes the
[use and advantages of Falcon](http://autodesk.typepad.com/bpa/2012/12/cfd-integrated-into-revit.html) in
some depth on his
[Building Performance Analysis](http://autodesk.typepad.com/bpa) blog
that I was previously unaware of.
He includes detailed explanations of the kind of problems this software addresses, getting started, and comparisons with other technologies such as the wind tunnel feature in
[Project Vasari](http://autodeskvasari.com) and
[Simulation CFD 360](http://usa.autodesk.com/adsk/servlet/pc/index?id=17141897&siteID=123112).
Fascinating stuff.

By the way, this is extremely interesting to look at for any Revit add-in developer, even if you do not care about this specific kind of analysis, because it shows an impressive example of using Revit as a front-end input tool to a powerful analysis component, reporting the results back graphically using the analysis visualisation framework AVF. If you hook up these components intelligently, a lot of functionality can be achieved with little effort.

#### Centroid and Volume of a Solid

Returning to the centroid and volume calculation, the Revit API provides both of these through the Solid ComputeCentroid method and Volume property.
As said, a conversation prompted me to explore how to implement these calculations on my own as well.

Calculating exact results for an arbitrary solid is not trivial, of course, and could currently only be achieved making use of an external library.

If high precision is not paramount, however, we can simplify the solid to a planar faceted representation by triangulating all its faces to create a
[polyhedral](http://en.wikipedia.org/wiki/Polyhedron) approximation of it.

Determining the centroid of a polyhedron is something that can be achieved in a very few lines of code, as I will show below.

#### Gap Free Triangulation for Polyhedral Approximation

Happily, the Revit 2013 API provides a method for triangulating the entire surface of a solid in one single call, ensuring a closed volume, unlike doing it face by face, which does not.

The Face.Triangulate method returns a triangular mesh approximation to the face.
Revit defines the approximation tolerance internally.
Calling it separately for neighbouring faces, however, will return independent triangulations that do not line up where the faces meet, leaving
[gaps in the shell](http://thebuildingcoder.typepad.com/blog/2010/12/birthdays-and-gaps-in-shells.html) of
the original solid.

This deficiency was eliminated by the SolidUtils class introduced in Revit 2013, which provides the
TessellateSolidOrShell method to facet an entire solid or open shell in one single call.
This enables each boundary component of the solid or shell to be represented by a single properly closed triangulated structure.

#### Polyhedron Centroid Calculation Algorithm

With these necessary and powerful basics in place, I searched the Internet for a bit of help on implementing an algorithm to calculate the centroid, and found a discussion on determining the
[centre of mass of a 3D model](http://gmc.yoyogames.com/index.php?showtopic=397122) from
the
[Game Maker Community](http://gmc.yoyogames.com).
Here is my edited version summarising the results of that thread:

**Question:** I'm working on a model editor and I have a set centre function to reset the centre of a model. I can do this a few ways which I have implemented.

**Method 1:** Add vertices, average them and move the model; pseudocode:

```
  for each point
    dx += point.x
    dy += point.y
    dz += point.z
    count ++

  dx/=count
  dy/=count
  dz/=count

  for each point
    point.x-=dx
    point.y-=dy
    point.z-=dz
```

The problem: If the model has many points in one area, the deviation favours that area.
Which is not always good because some models have many, many points like at the tip of a gun or the cone of a ship.

**Method 2:** Find min max of vertices, average the min max into a deviation and move the model:

```
  for each point
    mindx = min( point.x, mindx )
    maxdx = max( point.x, maxdx )
    same for y and z

  dx = (mindx + maxdx) / 2
  same for dy, dz;

  for each point
    point.x -= dx
    same for y and z
```

That works OK.

For my third method, I would really like to centre the model on its estimated centre mass.

I have a series of 3 points defining the planes/faces.
I figure I could plug either the plane area or perimeter into the deviation calculation.
That way many points defining a tiny area would not affect the calculation that much.
But I can't figure out the right math for this.

Can you determine the surface areas of each face?

**Answer:** Well, one would assume that we'd want to count the inside of the model into the mass calculation as well, in which case you want to find the volume of the model.
This actually isn't too difficult to compute assuming that the model is closed (no holes; otherwise it wouldn't really have a well-defined volume) and that it's simple (in the mathematical sense; that means it doesn't intersect with itself).
Each triangular face contributes a tetrahedron's worth of volume to the whole model.
To find this volume given the three vertices of the face v1, v2, and v3, you need only compute v1 · (v2 × v3 ) / 6.

The following is very important.
The vertices of every face must be oriented the same way.
This means that if you look at a face from the outside of a model, its vertices should be oriented in a clockwise fashion (counter-clockwise is valid too, as long as you're consistent).

So, now we essentially have a way to compute a weighting factor for every face in the model. The coordinates we have to weight this by are the centroids of each tetrahedron, given by (v1 + v2 + v3)/4.
This is because one of the vertices of every tetrahedron is the origin so it's 0, so there's no reason to include it here.
So, the centroid of the model is given by the following:

∑ (v1 + v2 + v3)/4 [v1 · (v2 × v3 )] / ∑ [v1 · (v2 × v3)]

The sum is performed over each face.

Also, the factor of 6 in the volume is left out because it cancels out because of the division.
So, loop over each face, compute the volume contribution and add that onto a total volume counter and also multiply it by that average position (you can move the division by 4 outside of the sum to make the loop faster; that way only one division needs to be performed instead of one for each face) of the tetrahedron (i.e. multiply it by the sum of the vertices).
Then, outside of the loop, divide by the total volume times 4 (assuming you moved that division by 4 outside of the loop).
You now have the centroid assuming constant density.

The variables are all vectors.
The product of the three vectors is a triple scalar product.
It's the dot product with the cross product (and also the determinant of the matrix of these vectors).
It's equal to this, which can be used as a formula for the volume:

```
  x1*(y2*z3 - y3*z2)
  + y1*(z2*x3 - z3*x2)
  + z1*(x2*y3 - x3*y2)
```

This is because one of the vertices of every tetrahedron is the origin.

Note that this does not require the model to be convex.
All the tetrahedrons are not necessarily entirely contained inside the model.

I only assume that the model is closed and simple.
Any tetrahedrons outside the model will have their area computed to be negative.
They'll subtract off their contributions.
The origin of the system can be anywhere as well.
It doesn't have to be inside the model.

Here is the entire pseudo-code:

```
  var cx, cy, cz, volume, v, i, x1, y1, z1, x2, y2, z2, x3, y3, z3;
  volume = 0;
  cx = 0; cy = 0; cz = 0;
  // Assuming vertices are in vertX[i], vertY[i], and vertZ[i]
  // and faces are faces[i, j] where the first index indicates the
  // face and the second index indicates the vertex of that face
  // The value in the faces array is an index into the vertex array
  i = 0;
  repeat (numFaces) {
    x1 = vertX[faces[i, 0]]; y1 = vertY[faces[i, 0]]; z1 = vertZ[faces[i, 0]];
    x2 = vertX[faces[i, 1]]; y2 = vertY[faces[i, 1]]; z2 = vertZ[faces[i, 1]];
    x3 = vertX[faces[i, 2]]; y3 = vertY[faces[i, 2]]; z3 = vertZ[faces[i, 2]];
    v = x1*(y2*z3 - y3*z2) + y1*(z2*x3 - z3*x2) + z1*(x2*y3 - x3*y2);
    volume += v;
    cx += (x1 + x2 + x3)*v;
    cy += (y1 + y2 + y3)*v;
    cz += (z1 + z2 + z3)*v;
    i += 1;
  }
  // Set centroid coordinates to their final value
  cx /= 4 * volume;
  cy /= 4 * volume;
  cz /= 4 * volume;
  // And, just in case you want to know the total volume of the model:
  volume /= 6;
```

Remember that the vertices of each face must be oriented in the same way (one way to see if they are is to turn on back face culling; if the model appears normally then they're all oriented properly--the model could also appear inside-out which would also be a valid orientation).

Part of this is actually calculating the area of each triangle without having to figure out the angles, simply using the 3 side lengths.

One way to achieve this is using Heron's Formula:

```
  A = sqrt( s * (s - a) * (s - b) * (s - c) )
```

Here a, b, c are the lengths of each side and s is the semi-perimeter, s = (a + b + c) / 2.

You can also just take the magnitude of the cross product of two edge vectors of the face and divide it by 2.
This means that you can find the area of a face as a side effect of determining its normal vector, just as I do in my
[GetSignedPolygonArea](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html) method.

#### Revit Implementation and CentroidVolume Class

I used the algorithm described above to implement a volume and centre of mass calculation for a Revit solid.

As said, curved solids are approximated by triangulating them.
The result is pretty precise for solids that are planar faceted to start with.

The code includes an assertion to ensure that the volume calculated as a side effect is not too far removed from the value provided by the Solid.Volume property.

Since the algorithm above calculates both the centroid and volume, I implemented the following little helper class to package the two together:

```csharp
  class CentroidVolume
  {
    public CentroidVolume()
    {
    }

    public XYZ Centroid { get; set; }
    public double Volume { get; set; }

    override public string ToString()
    {
      return RealString( Volume ) + "@"
        + PointString( Centroid );
    }
  }
```

#### Polyhedron Centroid Calculation Implementation

My translation of the algorithm described above to Revit API looks like this, including the validation assertions at the end and an exception handler around the call to TessellateSolidOrShell for reasons explained below:

```csharp
CentroidVolume GetCentroid( Solid solid )
{
  CentroidVolume cv = new CentroidVolume();
  double v;
  XYZ v0, v1, v2;

  SolidOrShellTessellationControls controls
    = new SolidOrShellTessellationControls();

  controls.LevelOfDetail = 0;

  TriangulatedSolidOrShell triangulation = null;

  try
  {
    triangulation
      = SolidUtils.TessellateSolidOrShell(
        solid, controls );
  }
  catch( Autodesk.Revit.Exceptions
    .InvalidOperationException )
  {
    return null;
  }

  int n = triangulation.ShellComponentCount;

  for( int i = 0; i < n; ++i )
  {
    TriangulatedShellComponent component
      = triangulation.GetShellComponent( i );

    int m = component.TriangleCount;

    for( int j = 0; j < m; ++j )
    {
      TriangleInShellComponent t
        = component.GetTriangle( j );

      v0 = component.GetVertex( t.VertexIndex0 );
      v1 = component.GetVertex( t.VertexIndex1 );
      v2 = component.GetVertex( t.VertexIndex2 );

      v = v0.X\*(v1.Y\*v2.Z - v2.Y\*v1.Z)
        + v0.Y\*(v1.Z\*v2.X - v2.Z\*v1.X)
        + v0.Z\*(v1.X\*v2.Y - v2.X\*v1.Y);

      cv.Centroid += v \* (v0 + v1 + v2);
      cv.Volume += v;
    }
  }

  // Set centroid coordinates to their final value

  cv.Centroid /= 4 \* cv.Volume;

  XYZ diffCentroid = cv.Centroid
    - solid.ComputeCentroid();

  Debug.Assert( 0.6 > diffCentroid.GetLength(),
    "expected centroid approximation to be "
    + "similar to solid ComputeCentroid result" );

  // And, just in case you want to know
  // the total volume of the model:

  cv.Volume /= 6;

  double diffVolume = cv.Volume - solid.Volume;

  Debug.Assert( 0.3 > Math.Abs(
    diffVolume / cv.Volume ),
    "expected volume approximation to be "
    + "similar to solid Volume property value" );

  return cv;
}
```

As you may have noticed, the assertion limits are pretty arbitrarily chosen.

#### Support for Multiple Solids

Since the centroid of any given element will be determined by all of its solids, which may be multiple, I implemented the following GetCentroid method to determine all solids belonging to a given element and calculate its total centroid and volume across all its component solids:

```python
/// <summary>
/// Calculate centroid for all non-empty solids
/// found for the given element. Family instances
/// may have their own non-empty solids, in which
/// case those are used, otherwise the symbol geometry.
/// The symbol geometry could keep track of the
/// instance transform to map it to the actual
/// project location. Instead, we ask for
/// transformed geometry to be returned, so the
/// resulting solids are already in place.
/// </summary>
CentroidVolume GetCentroid(
  Element e,
  Options opt )
{
  CentroidVolume cv = null;

  GeometryElement geo = e.get\_Geometry( opt );

  Solid s;

  if( null != geo )
  {
    // List of pairs of centroid, volume for each solid

    List<CentroidVolume> a
      = new List<CentroidVolume>();

    Document doc = e.Document;

    if( e is FamilyInstance )
    {
      geo = geo.GetTransformed(
        Transform.Identity );
    }

    GeometryInstance inst = null;

    CentroidVolume cv1;

    foreach( GeometryObject obj in geo )
    {
      s = obj as Solid;

      if( null != s
        && 0 < s.Faces.Size
        && SolidUtils.IsValidForTessellation( s )
        && (null != ( cv1 = GetCentroid( s ) ) ) )
      {
        a.Add( cv1 );
      }
      inst = obj as GeometryInstance;
    }

    if( 0 == a.Count && null != inst )
    {
      geo = inst.GetSymbolGeometry();

      foreach( GeometryObject obj in geo )
      {
        s = obj as Solid;

        if( null != s
          && 0 < s.Faces.Size
          && SolidUtils.IsValidForTessellation( s )
          && (null != ( cv1 = GetCentroid( s ) ) ) )
        {
          a.Add( cv1 );
        }
      }
    }

    // Get the total centroid from the partial
    // contributions. Each contribution is weighted
    // with its associated volume, which needs to
    // be factored out again at the end.

    if( 0 < a.Count )
    {
      cv = new CentroidVolume();
      foreach( CentroidVolume cv2 in a )
      {
        cv.Centroid += cv2.Volume \* cv2.Centroid;
        cv.Volume += cv2.Volume;
      }
      cv.Centroid /= a.Count \* cv.Volume;
    }
  }
  return cv;
}
```

The calculation of the total volume and centroid is similar to the base algorithm described above, with each centroid contribution weighted by the corresponding volume.

#### Mainline with Picking Twists

Finally, here is the external command Execute method implementation driving this and including some additional useful picking interaction handling twists, such as:

- Support for pre-selection or interactive picking.
- Checking for a valid active view type before prompting the user to pick an element.
- Exception handling to gracefully terminate picking and the entire command on cancels.

The resulting code looks like this:

```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;
  List<ElementId> ids = new List<ElementId>();
  Selection sel = uidoc.Selection;
  SelElementSet set = sel.Elements;

  if( 0 < set.Size )
  {
    foreach( Element e in set )
    {
      ids.Add( e.Id );
    }
  }
  else
  {
    if( ViewType.Internal == doc.ActiveView.ViewType )
    {
      TaskDialog.Show( \_caption,
        "Cannot pick elements in this view: "
        + doc.ActiveView.Name );

      return Result.Failed;
    }

    try
    {
      IList<Reference> refs = sel.PickObjects(
        ObjectType.Element,
        "Please select some elements" );

      foreach( Reference r in refs )
      {
        ids.Add( r.ElementId );
      }
    }
    catch( Autodesk.Revit.Exceptions.OperationCanceledException )
    {
      return Result.Cancelled;
    }
  }

  Options opt = app.Create.NewGeometryOptions();

  foreach( ElementId id in ids )
  {
    Element e = doc.GetElement( id );

    CentroidVolume cv = GetCentroid( e, opt );

    Debug.Print( "{0} {1}",
      (null == cv ? "<nil>" : cv.ToString()),
      ElementDescription( e ) );
  }
  return Result.Succeeded;
}
```

#### Results

I ran this on the basic sample model delivered with Revit, rac\_basic\_sample\_project.rvt, simply selecting all elements on level 1.
Not surprisingly, this causes a couple of hiccups, notably a few assertions stating that my volume approximation does not match Revit's and an exception thrown by the TessellateSolidOrShell method, in spite of calling IsValidForTessellation beforehand to ensure tht I only feed in acceptable solids to it.
This exception is caused by the teapot in the model, which is in fact a rather abnormal building element, but still...

These hiccups can be ironed out.
Here is the report logged to the debug output window with the hiccups and exception messages manually edited away, informing us that a large number of elements in the model are correctly processed and that for all of them my calculated approximate values correspond more or less to the ones reported by the built-in Revit API methods;
copy and paste to an editor to see the truncated lines in full:

```
115.87@(-52.42,8.9,11.04) Wall Walls <117653 Foundation - 305 Concrete>
455.96@(-20.92,6.23,11.54) Wall Walls <117654 Generic - 200>
461.1@(-24.09,27.23,11.75) Wall Walls <117698 CORR>
<nil> Wall Walls <117714 Storefront>
52.65@(-19.42,13.54,4.88) Wall Walls <117945 Interior - Partition>
36.09@(-12.55,15.23,5.22) Wall Walls <117963 Interior - Partition>
<nil> Element Stairs <118049 open treads>
<nil> Railing Railings <118116 Handrail - Pipe>
64.4@(-7.56,11.37,4.92) Wall Walls <118356 Interior - Partition>
168.87@(11.08,17.15,13.97) Wall Walls <123926 CORR>
<nil> Wall Walls <123967 Storefront>
0.32@(5.54,12.96,0.69) FamilyInstance Doors Window_Insert <123981 Window_Insert>
0.24@(11.08,25.92,0.13) Mullion Curtain Wall Mullions Rectangular Mullion <123986 64 x 128 rectangular>
0.29@(11.08,22.37,1.31) Mullion Curtain Wall Mullions Rectangular Mullion <123987 64 x 128 rectangular>
0.34@(5.54,12.96,1.97) FamilyInstance Doors Window_Insert <123990 Window_Insert>
0.29@(10.95,27.23,1.31) Mullion Curtain Wall Mullions Rectangular Mullion <123995 64 x 128 rectangular>
0.29@(11.08,22.37,3.94) Mullion Curtain Wall Mullions Rectangular Mullion <124013 64 x 128 rectangular>
0.29@(11.08,27.1,1.31) Mullion Curtain Wall Mullions Rectangular Mullion <124015 64 x 128 rectangular>
0.29@(11.08,27.1,3.94) Mullion Curtain Wall Mullions Rectangular Mullion <124016 64 x 128 rectangular>
0.24@(11.08,25.92,2.62) Mullion Curtain Wall Mullions Rectangular Mullion <124017 64 x 128 rectangular>
0.29@(10.95,27.23,3.94) Mullion Curtain Wall Mullions Rectangular Mullion <124019 64 x 128 rectangular>
0.6@(-10.02,6.82,1.5) FamilyInstance Windows Fixed <125448 406 x 610>
3.16@(2.85,4.18,0.91) FamilyInstance Doors Double-Glass 1 <126295 1800 x 2100>
252.05@(-24.2,34.28,6.92) Wall Walls <127132 Foundation - 305 Concrete>
236.66@(-1.03,9.27,2.21) FamilyInstance Walls FIREPLACE <127491 FIREPLACE>
51.52@(-31.92,13.3,4.86) Wall Walls <127659 Interior - Partition>
79.29@(-31.83,20.23,5.11) Wall Walls <127660 Interior - Partition>
3.84@(-11.31,6.72,1.2) FamilyInstance Doors Single-Flush <127661 800 x 2100>
3.84@(-9.97,6.72,1.2) FamilyInstance Doors Single-Flush <127662 800 x 2100>
17.84@(-35.67,24.22,5.51) Wall Walls <127663 Interior - Partition>
0.26@(-5.33,3.72,0.31) FamilyInstance Plumbing Fixtures Sink Vanity-Square <127666 508 x 445>
3.84@(-11.92,7.41,1.2) FamilyInstance Doors Single-Flush <127667 800 x 2100>
1.47@(-8.6,1.54,1.38) FamilyInstance Windows Fixed <127718 1219 X 915>
1.47@(-5.51,1.54,1.38) FamilyInstance Windows Fixed <127719 1219 X 915>
28.59@(-27.64,23.9,6.44) Wall Walls <127941 Interior - Partition>
11.78@(-19.42,25.25,4.95) Wall Walls <128006 Interior - Partition>
<nil> CurtainGridLine Curtain Wall Grids <128399 Grid Line>
0.29@(0.69,13.62,0.69) FamilyInstance Doors Window_Insert <128400 Window_Insert>
0.3@(0.69,13.62,1.97) FamilyInstance Doors Window_Insert <128401 Window_Insert>
<nil> CurtainGridLine Curtain Wall Grids <128408 Grid Line>
0.57@(2.29,13.62,0.69) FamilyInstance Doors Window_Insert <128409 Window_Insert>
0.59@(2.29,13.62,1.97) FamilyInstance Doors Window_Insert <128410 Window_Insert>
0.46@(4.58,27.23,0.13) Mullion Curtain Wall Mullions Rectangular Mullion <128432 64 x 128 rectangular>
0.2@(1.37,27.23,2.62) Mullion Curtain Wall Mullions Rectangular Mullion <128433 64 x 128 rectangular>
0.46@(4.58,27.23,2.62) Mullion Curtain Wall Mullions Rectangular Mullion <128434 64 x 128 rectangular>
0.29@(2.41,27.23,1.31) Mullion Curtain Wall Mullions Rectangular Mullion <128450 64 x 128 rectangular>
0.29@(2.41,27.23,3.94) Mullion Curtain Wall Mullions Rectangular Mullion <128452 64 x 128 rectangular>
4.73@(19.5,10.48,9.25) FamilyInstance Columns Rectangular Column <128571 140 X 140>
2.38@(19.5,6.82,3.67) FamilyInstance Columns Rectangular Column <128683 140 X 140>
<nil> CurtainGridLine Curtain Wall Grids <130207 Grid Line>
0.55@(4.42,13.62,0.69) FamilyInstance Doors Window_Insert <130208 Window_Insert>
0.58@(4.42,13.62,1.97) FamilyInstance Doors Window_Insert <130209 Window_Insert>
0.44@(8.85,27.23,0.13) Mullion Curtain Wall Mullions Rectangular Mullion <130248 64 x 128 rectangular>
0.44@(8.85,27.23,2.62) Mullion Curtain Wall Mullions Rectangular Mullion <130249 64 x 128 rectangular>
0.29@(6.75,27.23,1.31) Mullion Curtain Wall Mullions Rectangular Mullion <130256 64 x 128 rectangular>
0.29@(6.75,27.23,3.94) Mullion Curtain Wall Mullions Rectangular Mullion <130257 64 x 128 rectangular>
2.38@(-8.38,34.98,3.67) FamilyInstance Columns Rectangular Column <130550 140 X 140>
4.23@(-5.81,5.05,1.21) FamilyInstance Doors Single-Flush <135477 915 x 2134>
13.47@(-15.67,13.3,4.92) Wall Walls <135527 Interior - Partition>
8.72@(-1.14,2.81,0.13) FamilyInstance Furniture Chair-Corbu <137191 Chair-Corbu>
8.72@(0.4,2.99,0.13) FamilyInstance Furniture Chair-Corbu <137272 Chair-Corbu>
4.73@(-26.85,34.98,9.25) FamilyInstance Columns Rectangular Column <137480 140 X 140>
0.29@(0.37,27.23,1.31) Mullion Curtain Wall Mullions Rectangular Mullion <139162 64 x 128 rectangular>
0.29@(0.37,27.23,3.94) Mullion Curtain Wall Mullions Rectangular Mullion <139163 64 x 128 rectangular>
0.2@(1.37,27.23,0.13) Mullion Curtain Wall Mullions Rectangular Mullion <139229 64 x 128 rectangular>
9.04@(-35.67,18.87,4.92) Wall Walls <140108 Interior - Partition>
15.8@(-39.9,17.9,6.41) Wall Walls <140130 Interior - Partition>
4.39@(-5.72,2.54,0.51) FamilyInstance Doors Bifold-4 Panel <140140 1800 x 2100>
9.04@(-28.17,18.87,4.92) Wall Walls <140190 Interior - Partition>
15.93@(-23.79,17.9,6.27) Wall Walls <140191 Interior - Partition>
4.39@(-3.4,2.54,0.51) FamilyInstance Doors Bifold-4 Panel <140192 1800 x 2100>
33.57@(1.08,8.95,4.79) Wall Walls <140204 Generic - 200>
13.99@(-8.49,6.36,0.37) FamilyInstance Specialty Equipment Dryer <144101 686 x 635 x 889>
13.63@(-10.27,8.48,0.49) FamilyInstance Specialty Equipment Washer <144494 686 x 635 x 889>
4.39@(-4.57,3.4,0.51) FamilyInstance Doors Bifold-4 Panel <144534 1800 x 2100>
4.39@(-3.27,3.4,0.51) FamilyInstance Doors Bifold-4 Panel <144657 1800 x 2100>
10.42@(-27.54,25.45,5.03) Wall Walls <144706 Interior - Partition>
64.17@(5.93,7.23,4.92) Wall Walls <144998 CORR>
22.4@(10.08,9.17,4.92) Wall Walls <145015 CORR>
12.23@(10.58,11.11,4.92) Wall Walls <145032 CORR>
36@(6.75,10.69,2) FamilyInstance Casework Bar <145182 Bar>
6.5@(-7.92,13.3,2.38) Wall Walls <145199 Interior - Partition>
0.29@(0.34,27.23,3.94) Mullion Curtain Wall Mullions Rectangular Mullion <150839 64 x 128 rectangular>
0.29@(0.34,27.23,1.31) Mullion Curtain Wall Mullions Rectangular Mullion <150840 64 x 128 rectangular>
308.55@(-22,6.39,-0.49) Wall Walls <150987 Generic - 305>
134.41@(-44.42,17.07,1.02) Wall Walls <151006 Generic - 200>
256.5@(-25.18,27.08,-0.49) Wall Walls <151081 Generic - 305>
1284.19@(-38.12,13.27,4) Room Rooms <152552 BEDROOM 1 5>
<nil> RoomTag Room Tags <152553 Room Tag>
1298.34@(-25.66,13.27,4) Room Rooms <152555 BEDROOM 2 6>
<nil> RoomTag Room Tags <152556 Room Tag>
420.33@(-27.44,22.07,4) Room Rooms <152557 HALL 7>
<nil> RoomTag Room Tags <152558 Room Tag>
423.9@(-39.98,23.65,4) Room Rooms <152559 BATH 8>
<nil> RoomTag Room Tags <152560 Room Tag>
839.97@(-10.26,9.39,4) Room Rooms <152561 STORAGE 9>
<nil> RoomTag Room Tags <152562 Room Tag>
<nil> ModelLine <Room Separation> <152596 Model Lines>
352.08@(-27.54,25.49,4) Room Rooms <152642 MECH 11>
<nil> RoomTag Room Tags <152643 Room Tag>
19.63@(-6.85,1.82,0.17) FamilyInstance Furniture Bed-Box <153466 Double 1346 x 1880>
19.63@(-3.76,1.79,0.17) FamilyInstance Furniture Bed-Box <153556 Double 1346 x 1880>
<nil> ReferencePlane Reference Planes <165634 Reference Plane>
<nil> Opening Rectangular Straight Wall Opening <166134 Rectangular Straight Wall Opening>
0.47@(-2.86,22.65,0.01) FamilyInstance Furniture Rug_zk <167847 1220 x 2134>
2.44@(-0.57,4.53,0.23) FamilyInstance Furniture Table-Coffee <168561 915 x 1830 x 457>
0.02@(-1.7,11.35,0.98) FamilyInstance Generic Models teapot <169568 teapot>
0.1@(-2.09,6.25,0.48) FamilyInstance Lighting Fixtures Floor Lamp 1 <171113 150 watt Halogen>
3@(-14.28,8.03,0.18) FamilyInstance Plumbing Fixtures Tub-Rectangular-3D <178179 Tub-Rectangular-3D>
<nil> CurtainGridLine Curtain Wall Grids <178753 Grid Line>
0.39@(11.19,23.55,1.38) Panel Curtain Panels System Panel <178754 Glazed>
0.41@(11.19,23.55,3.94) Panel Curtain Panels System Panel <178755 Glazed>
0.24@(11.08,23.55,0.13) Mullion Curtain Wall Mullions Rectangular Mullion <178774 64 x 128 rectangular>
0.24@(11.08,23.55,2.62) Mullion Curtain Wall Mullions Rectangular Mullion <178775 64 x 128 rectangular>
0.29@(11.08,24.73,1.31) Mullion Curtain Wall Mullions Rectangular Mullion <178782 64 x 128 rectangular>
0.29@(11.08,24.73,3.94) Mullion Curtain Wall Mullions Rectangular Mullion <178783 64 x 128 rectangular>
<nil> Room Rooms <180268 LIVING ROOM 12>
<nil> RoomTag Room Tags <180269 Room Tag>
11.72@(-0.33,2.33,0.13) FamilyInstance Furniture Chair-Corbu <181500 Couch Corbu>
```

This is more than enough for me and my rather quick and dirty exploration of this.
Improved filtering and error handling is left as an exercise to the inclined reader.

#### Download

Here is
[GetCentroid.zip](zip/GetCentroid.zip) containing
the complete source code, Visual Studio solution and add-in manifest for this command.