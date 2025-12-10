---
post_number: "1387"
title: "Wall Openings"
slug: "wall_openings"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'python', 'references', 'revit-api', 'sheets', 'views', 'walls', 'windows']
source_file: "1387_wall_openings.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1387_wall_openings.html"
---

### Retrieving Wall Openings and Sorting Points
I continue my rather active involvement in the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160).
Lets look at one of them, and an associated point or two:
- [Retrieving wall openings](#2)
- [Ray shooting solution](#2.1)
- [CmdWallOpenings implementation](#3)
- [Addendum](#3.1)
- [Faster lexicographical point sorting](#4)
#### Retrieving Wall Openings
A very nice geometric issue was raised yesterday by Eirik Aasved Holst concerning the [determination of wall openings](http://forums.autodesk.com/t5/revit-api/get-wall-openings/m-p/5955539):
\*\*Question:\*\* I'm struggling to get wall openings using the API. What I'm interested in is the coordinates of a rectangular wall opening:
![Wall opening points](img/wall_openings_01.jpeg)
I've tried FindInserts(), but it's not given that anything is inserted in the opening (cf. the [addendum below](#3.1)).
Does anyone have a solution for this?
\*\*Answer:\*\* This is an interesting question, and therefore has many answers.
You can determine the elements inserted in the wall (incl. openings) by temporarily deleting the wall. That gives you a list of all the dependent objects that also got deleted. Then you have the elements you are after.
You could look at the wall geometry before and after deleting the inserted elements and determine the differences.
You could query the wall for its geometry and determine all the holes in the elevation view.
My personal favourite candidate right now, off the hip, would be to shoot a ray along the wall centre line and determine all the faces that it hits.
That only works if you know one specific height or elevation (or several) at which all the openings really are open, so that they really have side faces that the ray intersection will hit.
This technique is demonstrated to find columns intersecting a wall by the FindReferencesByDirection / FindColumns SDK sample.
Have fun exploring, and please let us know what you end up with.
#### Ray Shooting Solution
\*\*Response:\*\* Wonderful! The ray shooting solution works perfectly.
I use the function for a planar view and set the ray elevation equal to view elevation.
For anyone else interested in using the same technique to locate wall openings, remember to set the ray origin a step outside the wall to determine if the wall starts with an opening or not:
![Wall openings at wall ends](img/wall_openings_03.jpeg)
Pseudo-code for my solution:

```
  WallOpening2D - a simple class with two
    coordinates and some other basic info.

  List GetWallOpenings(
    Wall wall,
    ViewPlan view )
  {
    var rayStart = new XYZ(
      wallOrigin.X - wallDirection.X,
      wallOrigin.Y - wallDirection.Y,
      GetElevation(view));

    pointList = (from reference in
      new ReferenceIntersector(view)
        .Find(rayStart, wallDirection)
        .where(IsSurface)
        .where(ref => ref.Proximity
          < wallLength + "step outside"))
        select reference.GetReference.GlobalPoint)

    if(!pointList.First().IsAlmostEqualTo(wallOrigin) //CHECK IF FIRST POINT IS NOT AT WALL START
      pointList.Insert(0, wallOrigin);
    else
      pointList.remove(pointList(First));

    if(!IsEven(pointList.Count)  //IF NOT EVEN POINTCOUNT - WALL ENDS WITH OPENING
      pointList.Add(wallEndPoint);

    var wallOpenings = new List();
    for(i = 0; i < pointList.Count -1; i += 2)
      wallOpenings.Add(new WallOpening2d(pointList[i], pointList[i+1]);

    return wallOpenings;
  }
```

\*\*Answer:\*\* I am very glad it helped.
Thank you for sharing the pseudo code.
I started implementing the real thing based on that and ran into one single little snag:
- error CS1502: The best overloaded method match for 'Autodesk.Revit.DB.ReferenceIntersector.ReferenceIntersector(Autodesk.Revit.DB.View3D)' has some invalid arguments
- error CS1503: Argument 1: cannot convert from 'Autodesk.Revit.DB.ViewPlan' to 'Autodesk.Revit.DB.View3D'
Here is my implementation so far:

```
List GetWallOpenings(
  Wall wall,
  ViewPlan view )
{
  Document doc = wall.Document;
  Level level = doc.GetElement( view.LevelId ) as Level;
  double elevation = level.Elevation;
  Curve c = (wall.Location as LocationCurve).Curve;
  XYZ wallOrigin = c.GetEndPoint(0);
  XYZ wallEndPoint = c.GetEndPoint(1);
  XYZ wallDirection =wallEndPoint - wallOrigin;
  double wallLength = wallDirection.GetLength();
  wallDirection = wallDirection.Normalize();
  UV offset = new UV( wallDirection.X, wallDirection.Y );
  double step_outside = offset.GetLength();

  XYZ rayStart = new XYZ( wallOrigin.X - offset.U,
    wallOrigin.Y - offset.V, elevation );

  ReferenceIntersector intersector = new ReferenceIntersector( view ); // *** error here ***

  IList refs = intersector.Find( rayStart, wallDirection );
  List pointList = new List( refs
    .Where( r => IsSurface(r.GetReference()))
    .Where( r => r.Proximity < wallLength + step_outside)
    .Select( r => r.GetReference().GlobalPoint) );

  // Check if first point is not at wall start.
  // If so, the wall begins with an opening, so
  // add its start point.

  if(!pointList.First().IsAlmostEqualTo(wallOrigin))
  {
    pointList.Insert(0, wallOrigin);
  }
  else
  {
    pointList.Remove(pointList.First());
  }

  // If the point count in not even, the wall
  // ends  with an opening, so add its end as
  // a new last point.

  if(!IsEven(pointList.Count))
  {
    pointList.Add(wallEndPoint);
  }

  int n = pointList.Count;
  var wallOpenings = new List( n / 2 );
  for( int i = 0; i < n; i += 2 )
  {
    wallOpenings.Add( new WallOpening2d {
      Start = pointList[i],
      End = pointList[i + 1] } );
  }
  return wallOpenings;
}
```

I obviously have to provide a 3D view, not a plan one.
I also get a pretty weird point list from my sample wall with four openings, none of them at either end of the wall, all in the middle.
The wall start point appears twice, the wall end point does not appear at all, and the points are not sorted by proximity:
![Intersection points returned by ReferenceIntersector.Find](img/wall_openings_04.png)
I have to add some clean-up and fool-proofing to get a reliable result.
I implemented a working command
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2016.0.124.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.124.0).
The external command name
is [CmdWallOpenings](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdWallOpenings.cs).
\*\*Response:\*\* Some corrections to my pseudocode:
- You are right about the view3D. I'm using the 2D view to get the elevation of the view range cut plane. For the ReferenceIntersector, I'm using the standard 3D view (just make sure it does not have an active section box and that all relevant elements are visible).

```
  var default3DView
    = new FilteredElementCollector(doc)
      .OfClass(typeof (View3D))
      .ToElements()
      .Cast()
      .FirstOrDefault( v
        => v != null
        && !v.IsTemplate
        && v.Name.Equals("{3D}"));
```

Of course one may create a new 3D view if that works better.
- I forgot to specify that the reference intersector should only return Faces belonging to the relevant wall.

```
  var referenceIntersector
    = new ReferenceIntersector( wall.Id,
      FindReferenceTarget.Face, default3DView);
```

- "The wall start point appears twice, the wall end point does not appear at all, and the points are not sorted by proximity:"
The points will get sorted by proximity when the wall.id is specified to the ReferenceIntersector.
Your points are sorted by element, then by proximity.
To get the wall end point, it is important to set the "step_outside" sufficiently large in the clause

```
  .Where( r => r.Proximity < wallLength + step_outside)
```

The first point appears twice probably because `!pointList.First().IsAlmostEqualTo(wallOrigin)` in

```
  if(!pointList.First().IsAlmostEqualTo(wallOrigin))
  {
    pointList.Insert(0, wallOrigin);
  }
```

will always return true (or !false) and therefore the start point gets inserted.
This is because the wall origin Z coordinate is not necessarily the same as the ray start Z coordinate.
I changed the code to:

```
  Curve c = (wall.Location as LocationCurve).Curve;

  var wallStartPoint = new XYZ( c.GetEndPoint(0).X,
    c.GetEndPoint(0).Y, elevation);

  var rayStart = new XYZ(
    wallStartPoint.X - wallLine.Direction.X,
    wallStartPoint.Y - wallLine.Direction.Y,
    wallStartPoint.Z);

  ...

  if (!pointList.First().IsAlmostEqualTo(wallStartPoint))
    pointList.Insert(0, wallStartPoint);
```

My code works quite well now; thank you for the suggestion to use a shooting ray :-)
\*\*Answer:\*\* Thank you for picking up my questions.
As you can see from my later post and GitHub submission, I also found and solved all the issues you mention my own way.
1. I just use the currently active user selected 3D view.
2. Yes, I noticed and added that as well.
3. Nope. The points I list above are the exact results of the call to Find, with nothing added or removed. I think my problems were caused by using elevation zero, so the ray was passing through the bottom edges of the wall faces. That way, it is hit or miss whether an intersection is found or not. I raised it off the floor and extend it beyond the wall at each end by 0.1 feet, and then I get reliable results. I am surprised that is not causing problems for you.
4. My code works well too. Check it out on GitHub. If you see any further possible improvements, please let me know.
\*\*Suggestion 1:\*\* One addition concerning the default 3D view.
In a workshared environment, there is no "{3D}".
Instead of this, if a user creates a new default 3D view, this will be named like "{3d - username}".
Note the username suffix and the lowercase "d".
\*\*Suggestion 2:\*\* Instead of just elevating the one and single ray, what about using multiple rays, like a comb?
\*\*Answer:\*\* I totally agree.
Define a parameter specifying the minimum opening size, and then fan a comb up the entire wall side.
Retrieve all the resulting intersection points, sort them by proximity, eliminate duplicates, and Bob's your uncle.
#### CmdWallOpenings Implementation
As said, I implemented and tested this algorithm in a new [external command CmdWallOpenings](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdWallOpenings.cs)
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).
I cleaned it up a bit more, and the version presented below is
[release 2016.0.125.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.125.1).
I performed minimal testing of it on these three different walls:
![Three walls with openings](img/wall_openings_05.png)
Each of them has four openings.
Running the CmdWallOpenings command once for each wall reports the four openings like this:

```
  4 openings found:
  ((-0.42,18.27,0.1)-(2.59,18.27,0.1))
  ((5.49,18.27,0.1)-(8.49,18.27,0.1))
  ((9.43,18.27,0.1)-(12.43,18.27,0.1))
  ((13.36,18.27,0.1)-(16.37,18.27,0.1))
  4 openings found:
  ((0.78,4.77,0)-(2.73,5.68,0.1))
  ((4.47,6.5,0.1)-(7.19,7.76,0.1))
  ((8.93,8.58,0.1)-(11.66,9.84,0.1))
  ((13.1,10.52,0.1)-(15.82,11.79,0.1))
  4 openings found:
  ((2.23,-3.24,0.1)-(5.12,-2.41,0.1))
  ((6.96,-1.88,0.1)-(9.85,-1.06,0.1))
  ((12.32,-0.35,0.1)-(15.21,0.48,0.1))
  ((15.79,0.65,0.1)-(17.24,1.06,0))
```

![Wall opening coordinates](img/wall_openings_06_msg.png)
The wall opening data currently just includes the opening start and end points:

```
  ///
  /// A simple class with two coordinates
  /// and some other basic info.
  ///
  class WallOpening2d
  {
    //public ElementId Id { get; set; }
    public XYZ Start { get; set; }
    public XYZ End { get; set; }
    override public string ToString()
    {
      return "("
        //+ Id.ToString() + "@"
        + Util.PointString( Start ) + "-"
        + Util.PointString( End ) + ")";
    }
  }
```

I use an offset to raise the ray above the floor level and thus avoid intersecting the bottom edge of the wall faces:
This offset is also applied to lengthen the ray beyond the wall extents at both ends.

```
  ///
  /// Move out of wall and up from floor a bit
  ///
  const double _offset = 0.1; // feet
```

The following two predicate methods determine whether a number is even and whether a reference applies to a surface:

```
  ///
  /// Predicate: is the given number even?
  ///
  static bool IsEven( int i )
  {
    return 0 == i % 2;
  }

  ///
  /// Predicate: does the given reference refer to a surface?
  ///
  static bool IsSurface( Reference r )
  {
    return ElementReferenceType.REFERENCE_TYPE_SURFACE
      == r.ElementReferenceType;
  }
```

The latter could probably be eliminated, since we now specify that we are only interested in surfaces when calling the `ReferenceIntersector` `Find` method.
I ensure that distinct points are retained and processed by implementing this XYZ equality comparer and passing it into the LINQ `Distinct` method:

```
  class XyzEqualityComparer : IEqualityComparer<XYZ>
  {
    public bool Equals( XYZ a, XYZ b )
    {
      return Util.IsEqual( a, b );
    }

    public int GetHashCode( XYZ a )
    {
      return Util.PointString( a ).GetHashCode();
    }
  }
```

With those helper methods in place, the main `GetWallOpenings` method can be implemented as follows:

```
///
/// Retrieve all wall openings,
/// including at start and end of wall.
///
List<WallOpening2d> GetWallOpenings(
  Wall wall,
  View3D view )
{
  Document doc = wall.Document;
  Level level = doc.GetElement( wall.LevelId ) as Level;
  double elevation = level.Elevation;
  Curve c = ( wall.Location as LocationCurve ).Curve;
  XYZ wallOrigin = c.GetEndPoint( 0 );
  XYZ wallEndPoint = c.GetEndPoint( 1 );
  XYZ wallDirection = wallEndPoint - wallOrigin;
  double wallLength = wallDirection.GetLength();
  wallDirection = wallDirection.Normalize();
  UV offsetOut = _offset * new UV( wallDirection.X, wallDirection.Y );

  XYZ rayStart = new XYZ( wallOrigin.X - offsetOut.U,
    wallOrigin.Y - offsetOut.V, elevation + _offset );

  ReferenceIntersector intersector
    = new ReferenceIntersector( wall.Id,
      FindReferenceTarget.Face, view );

  IList<ReferenceWithContext> refs
    = intersector.Find( rayStart, wallDirection );

  // Extract the intersection points:
  // - only surfaces
  // - within wall length plus offset at each end
  // - sorted by proximity
  // - eliminating duplicates

  List<XYZ> pointList = new List<XYZ>( refs
    .Where<ReferenceWithContext>( r => IsSurface(
      r.GetReference() ) )
    .Where<ReferenceWithContext>( r => r.Proximity
      < wallLength + _offset + _offset )
    .OrderBy<ReferenceWithContext, double>(
      r => r.Proximity )
    .Select<ReferenceWithContext, XYZ>( r
      => r.GetReference().GlobalPoint )
    .Distinct<XYZ>( new XyzEqualityComparer() ) );

  // Check if first point is at the wall start.
  // If so, the wall does not begin with an opening,
  // so that point can be removed. Else, add it.

  XYZ q = wallOrigin + _offset * XYZ.BasisZ;

  bool wallHasFaceAtStart = Util.IsEqual(
    pointList[0], q );

  if( wallHasFaceAtStart )
  {
    pointList.RemoveAll( p
      => Util.IsEqual( p, q ) );
  }
  else
  {
    pointList.Insert( 0, wallOrigin );
  }

  // Check if last point is at the wall end.
  // If so, the wall does not end with an opening,
  // so that point can be removed. Else, add it.

  q = wallEndPoint + _offset * XYZ.BasisZ;

  bool wallHasFaceAtEnd = Util.IsEqual(
    pointList.Last(), q );

  if( wallHasFaceAtEnd )
  {
    pointList.RemoveAll( p
      => Util.IsEqual( p, q ) );
  }
  else
  {
    pointList.Add( wallEndPoint );
  }

  int n = pointList.Count;

  Debug.Assert( IsEven( n ),
    "expected an even number of opening sides" );

  var wallOpenings = new List<WallOpening2d>(
    n / 2 );

  for( int i = 0; i < n; i += 2 )
  {
    wallOpenings.Add( new WallOpening2d
    {
      Start = pointList[i],
      End = pointList[i + 1]
    } );
  }
  return wallOpenings;
}
```

The external command mainline Execute method drives it like this to achieve the following:
- Ensure we are running in a valid context.
- Select a wall, or retrieve the pre-selected one.
- Retrieve the wall openings.
- Report the result.

```
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  if( null == doc )
  {
    message = "Please run this command in a valid document.";
    return Result.Failed;
  }

  View3D view = doc.ActiveView as View3D;

  if( null == view )
  {
    message = "Please run this command in a 3D view.";
    return Result.Failed;
  }

  Element e = Util.SelectSingleElementOfType(
    uidoc, typeof( Wall ), "wall", true );

  List<WallOpening2d> openings = GetWallOpenings(
    e as Wall, view );

  int n = openings.Count;

  string msg = string.Format(
    "{0} opening{1} found{2}",
    n, Util.PluralSuffix( n ),
    Util.DotOrColon( n ) );

  Util.InfoMsg2( msg, string.Join(
    "\r\n", openings ) );

  return Result.Succeeded;
}
```

#### Addendum
Eirik's statement in the initial query implies that `FindInserts` does not return wall openings that do not host any insert, w.g., a door or window family instance.
This is not true, as Scott Wilson later pointed out. Scott also provided an alternative implementation determining and
displaying [wall opening profiles using FindInserts](http://thebuildingcoder.typepad.com/blog/2015/12/wall-opening-profiles-and-happy-holidays.html), which you might want to have a look at as well.
#### Faster Lexicographical Point Sorting
While evaluating different methods for sorting and comparing points, I also implemented the following `XyzProximityComparer`:

```
  class XyzProximityComparer : IComparer<XYZ>
  {
    XYZ _p;

    public XyzProximityComparer( XYZ p )
    {
      _p = p;
    }

    public int Compare( XYZ x, XYZ y )
    {
      double dx = x.DistanceTo( _p );
      double dy = y.DistanceTo( _p );
      return Util.IsEqual( dx, dy ) ? 0
        : ( dx < dy ? -1 : 1 );
    }
  }
```

I ended up not using it, for two reasons:
- The `ReferenceIntersector` already returns the points sorted by proximity.
- I can use a LINQ `OrderBy` clause instead.
Anyway, while researching this, I discovered the StackOverflow question
on [using IComparer for sorting](http://stackoverflow.com/questions/14336416/using-icomparer-for-sorting).
It discusses a point comparison class similar to `XyzEqualityComparer` presented above, only to continue with the following extremely interesting information:
> Note that if you target a .NET 3.5+ application you can use LINQ which is easier and even faster with sorting.
> LINQ version can be something like:
> var orderedList = Points.OrderBy(point => point.x)
> .ThenBy(point => point.y)
> .ToList();
This is definitely something to keep in mind next time you are thinking about sorting a collection of points!