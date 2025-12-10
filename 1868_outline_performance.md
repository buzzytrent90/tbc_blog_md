---
post_number: "1868"
title: "Outline Performance"
slug: "outline_performance"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'revit-api', 'sheets', 'walls']
source_file: "1868_outline_performance.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1868_outline_performance.html"
---

### High Performance Outline, Line-Plane Intersection
Today let's talk mainly about geometric analysis and performance:
- [High-performance outline optimisation](#2)
- [Solution](#2.1)
- [Simple line-plane intersection](#3)
- [Set base and survey clipped and unclipped](#4)
- [Two German uni BIM360 construction cloud startups](#5)
- [AI-based face streaming hits mainstream](#6)
#### High-Performance Outline Optimisation
If you are interested in high-performance use of the Revit API, you may be able to learn a trick or two from the StackOverflow discussion
on [how to get the bounding box for several elements](https://stackoverflow.com/questions/63083938/revit-api-how-can-i-get-bounding-box-for-several-elements).
\*\*Question:\*\* I need to find an outline for many elements (>100'000 items).
Target elements come from a `FilteredElementCollector`.
As usual, I'm looking for the fastest possible way.
For now, I tried to iterate over all elements to get its `BoudingBox.Min` and `BoudingBox.Max` and find out `minX`, `minY`, `minZ`, `maxX`, `maxY`, `maxZ`.
It works pretty accurately but takes too much time.
The problem described above is a part of a bigger one:
I need to find all the intersections of ducts, pipes and other curve-based elements from a linked model with walls, ceilings, columns, etc. in the general model and then place openings in the intersections.
I tried to use a combination of `ElementIntersectElement` filter and ` IntersectSolidAndCurve` method to find a part of curve inside element.
First, with an `ElementIntersectElement`, I tried to reduce a collection for further use of `IntersectSolidAndCurve`.
`IntersectSolidAndCurve` takes two arguments, solid and curve, and has to work in two nested one in the other loops.
So, it takes for 54000 walls (after filtering) and 18000 pipes, in my case, 972'000'000 operations.
With the number of operations 10 ^ 5, the algorithm shows an acceptable time.
I decided to reduce the number of elements by dividing the search areas by levels.
This works well for high-rise buildings, but is still bad for extended low structures.
I decided to divide the building by length, but I did not find a method that finds boundaries for several elements (the whole building).
I seem to be going in a wrong direction.
Is there are right way to achieve this with the Revit API?
\*\*Answer:\*\* In principle, what you describe is the proper approach and the only way to do it.
However, there may be many possibilities to optimise your code.
The Building Coder provides various utility functions that may help.
For instance, to [determine the bounding box of an entire family](https://thebuildingcoder.typepad.com/blog/2017/03/family-bounding-box-and-aec-hackathon-munich.html#3).
Many more in [The Building Coder samples `Util` module](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs).
Search there for "bounding box".
I am sure they can be further optimised as well for your case.
For instance, you may be able to extract all the `X` coordinates from all the individual elements' bounding box `Max` values and use a generic `Max` function to determine their maximum in one single call instead of comparing them one by one.
[Benchmark your code](https://thebuildingcoder.typepad.com/blog/2012/01/timer-code-for-benchmarking.html) to discover optimisation possibilities and analyse their effect on the performance.
\*\*Response:\*\* Thanks to Jeremy for advice and input on this issue.
I published my final result below and did some research on performance and accuracy.
The code in my answer processed in 3-5 seconds / 100'000 elements and works accurate in most cases.
However, there are cases where the `BoundingBoxIntersectsFilter` filters the item when it does not cross the `Outline`.
This happens if there is invisible geometry in the family.
There are other possible reasons that I have not yet found. More tests need to be done anyway.
\*\*Answer:\*\* Thank you very much for your appreciation and sharing your interesting code.
Using the built-in Revit filtering mechanisms will definitely be a lot faster than anything you can achieve in .NET, outside Revit memory.
However, I do not yet understand how you can use this to achieve the goal you describe above.
I thought you needed the collective bounding box of all elements.
You, however, seem to have an input variable of 500 meters and o be checking whether that contains all the elements.
Can you please explain the exact use of this algorithm, and the exact input and output data?
\*\*Response:\*\* Jeremy, you're absolutely right.
The minimum and maximum points in 3d-space are fed to the input to the method, so that all the elements are inside the outline built on that points, then we find the extreme points along `x` `y` `z`.
The output is 2 outline points.
#### Solution
To find boundaries, we can take advantage of the binary search idea.
The difference from the classic binary search algorithm is there is not an array, and we should find two numbers instead of one.
Elements in Geometry space could be presented as a 3-dimensional sorted array of `XYZ` points.
The Revit API provides an excellent `Quick Filter`, the `BoundingBoxIntersectsFilter`, that takes an instance of an `Outline`.
So, let’s define an area that includes all the elements for which we want to find the boundaries.
For my case, for example 500 meters, and create `min` and `max` point for the initial outline:
```csharp
double b = 500000 / 304.8;
XYZ min = new XYZ( -b, -b, -b );
XYZ max = new XYZ( b, b, b );
```
Below is an implementation for one direction; you can easily use it for three directions by calling and feeding the result of the previous iteration to the input:
```csharp
double precision = 10e-6 / 304.8;
var bb = new BinaryUpperLowerBoundsSearch(
doc, precision );
XYZ[] rx = bb.GetBoundaries( min, max, elems,
BinaryUpperLowerBoundsSearch.Direction.X );
rx = bb.GetBoundaries( rx[ 0 ], rx[ 1 ], elems,
BinaryUpperLowerBoundsSearch.Direction.Y );
rx = bb.GetBoundaries( rx[ 0 ], rx[ 1 ], elems,
BinaryUpperLowerBoundsSearch.Direction.Z );
```
The `GetBoundaries` method returns two `XYZ` points: lower and upper, which change only in the target direction; the other two dimensions remain unchanged:
```csharp
public class BinaryUpperLowerBoundsSearch
{
private Document doc;
private double tolerance;
private XYZ min;
private XYZ max;
private XYZ direction;
public BinaryUpperLowerBoundsSearch(
Document document, double precision )
{
doc = document;
this.tolerance = precision;
}
public enum Direction
{
X,
Y,
Z
}
///
/// Searches for an area that completely includes
/// all elements within a given precision.
/// The minimum and maximum points are used for the
/// initial assessment.
/// The outline must contain all elements.
/// summary>
/// The minimum point of the
/// BoundBox used for the first approximation.param>
/// The maximum point of the
/// BoundBox used for the first approximation.param>
/// Set of elementsparam>
/// The direction along which the
/// boundaries will be searchedparam>
/// Returns two points: first is the lower bound,
/// second is the upper boundreturns>
public XYZ[] GetBoundaries( XYZ minPoint, XYZ maxPoint,
ICollection elements, Direction axe )
{
// Since Outline is not derived from an Element class there
// is no possibility to apply transformation, so
// we have use as a possible directions only three vectors of basis
switch( axe )
{
case Direction.X:
direction = XYZ.BasisX;
break;
case Direction.Y:
direction = XYZ.BasisY;
break;
case Direction.Z:
direction = XYZ.BasisZ;
break;
default:
break;
}
// Get the lower and upper bounds as a projection
// on a direction vector
// Projection is an extention method
double lowerBound = minPoint.Projection( direction );
double upperBound = maxPoint.Projection( direction );
// Set the boundary points in the plane perpendicular
// to the direction vector.
// These points are needed to create BoundingBoxIntersectsFilter
// when IsContainsElements calls.
min = minPoint - lowerBound \* direction;
max = maxPoint - upperBound \* direction;
double[] res = UpperLower( lowerBound, upperBound, elements );
return new XYZ[ 2 ]
{
res[0] \* direction + min,
res[1] \* direction + max,
};
}
///
/// Check if there are any elements contains in
/// the segment [lower, upper]
/// summary>
/// True if any elements are in the segmentreturns>
private ICollection IsContainsElements( double lower,
double upper, ICollection ids )
{
var outline = new Outline( min + direction \* lower,
max + direction \* upper );
return new FilteredElementCollector( doc, ids )
.WhereElementIsNotElementType()
.WherePasses( new BoundingBoxIntersectsFilter( outline ) )
.ToElementIds();
}
private double[] UpperLower( double lower,
double upper, ICollection ids )
{
// Get the Midpoint for segment mid = lower + 0.5 \* (upper - lower)
var mid = Midpoint( lower, upper );
// Сheck if the first segment contains elements
ICollection idsFirst = IsContainsElements(
lower, mid, ids );
bool first = idsFirst.Any();
// Сheck if the second segment contains elements
ICollection idsSecond = IsContainsElements(
mid, upper, ids );
bool second = idsSecond.Any();
// If elements are in both segments
// then the first segment contains the lower border
// and the second contains the upper
// ---------\*\*|\*\*\*--------
if( first && second )
{
return new double[ 2 ]
{
Lower(lower, mid, idsFirst),
Upper(mid, upper, idsSecond),
};
}
// If elements are only in the first segment it contains both borders.
// We recursively call the method UpperLower until
// the lower border turn out in the first segment and
// the upper border is in the second
// ---\*\*\*\*\*---|-----------
else if( first && !second )
return UpperLower( lower, mid, idsFirst );
// Do the same with the second segment
// -----------|---\*\*\*\*\*---
else if( !first && second )
return UpperLower( mid, upper, idsSecond );
// Elements are out of the segment
// \*\* -----------|----------- \*\*
else
throw new ArgumentException(
"Segment does not contains elements. Try to make initial boundaries wider",
"lower, upper" );
}
///
/// Search the lower boundary of a segment containing elements
/// summary>
/// Lower boundaryreturns>
private double Lower( double lower, double upper,
ICollection ids )
{
// If the boundaries are within tolerance return lower bound
if( IsInTolerance( lower, upper ) )
return lower;
// Get the Midpoint for segment mid = lower + 0.5 \* (upper - lower)
var mid = Midpoint( lower, upper );
// Сheck if the segment contains elements
ICollection idsFirst = IsContainsElements(
lower, mid, ids );
bool first = idsFirst.Any();
// ---\*\*\*\*\*---|-----------
if( first )
return Lower( lower, mid, idsFirst );
// -----------|-----\*\*\*---
else
return Lower( mid, upper, ids );
}
///
/// Search the upper boundary of a segment containing elements
/// summary>
/// Upper boundaryreturns>
private double Upper( double lower, double upper,
ICollection ids )
{
// If the boundaries are within tolerance return upper bound
if( IsInTolerance( lower, upper ) )
return upper;
// Get the Midpoint for segment mid = lower + 0.5 \* (upper - lower)
var mid = Midpoint( lower, upper );
// Сheck if the segment contains elements
ICollection idsSecond = IsContainsElements(
mid, upper, ids );
bool second = idsSecond.Any();
// -----------|----\*\*\*\*\*--
if( second )
return Upper( mid, upper, idsSecond );
// ---\*\*\*\*\*---|-----------
else
return Upper( lower, mid, ids );
}
private double Midpoint( double lower, double upper )
=> lower + 0.5 \* (upper - lower);
private bool IsInTolerance( double lower, double upper )
=> upper - lower <= tolerance;
}
```
`Projection` is an extension method for the vector class to determine the length of projection of one vector onto another:
```csharp
public static class PointExt
{
public static double Projection(
this XYZ vector, XYZ other )
=> vector.DotProduct( other )
/ other.GetLength();
}
```
Many thanks to [Alexey Ovchinnikov](https://stackoverflow.com/users/9958255/alexey-ovchinnikov) for his impressive analysis and research and sharing this powerful and useful result.
#### Simple Line-Plane Intersection
Talking about geometric calculations and performance, I just took a quick look at a simple line-plane intersection algorithm this morning to answer
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to calculate the intersection between a plane and a penetrating line](https://forums.autodesk.com/t5/revit-api-forum/how-can-we-calculate-the-intersection-between-the-plane-and-the/m-p/9785834):
\*\*Question:\*\* I have a question about the way to calculate intersection point.
We have four points and their coordinates.
Let's assume we can create plane with these points.
When there is a line penetrating the plane, I guess there is an intersection point which the plane and the line meet:
![Line-plane intersection](img/line_plane_intersection.png "Line-plane intersection")
How can we calculate the coordinates of the intersection point?
\*\*Answer:\*\* You will only need three points to uniquely define the face, so the fourth point can actually be used to verify that all four are coplanar.
Calculating the intersection between a straight line and a plane is pretty easy, so the most efficient method to achieve this may possibly be to do it yourself:
- [Search the Internet for 'line plane intersect'](https://duckduckgo.com/?q=line+plane+intersect)
- [Read about line-plane intersection on Wikipedia](https://en.wikipedia.org/wiki/Line%E2%80%93plane_intersection)
- [Watch a YouTube video explaining the concepts](https://youtu.be/_W3aVWsMp14)
- Many algorithms in different languages to [find the intersection of a line with a plane](https://rosettacode.org/wiki/Find_the_intersection_of_a_line_with_a_plane)
- A StackOverflow discussion on [3D line-plane intersection](https://stackoverflow.com/questions/5666222/3d-line-plane-intersection)
If you prefer to use the official Revit API, you can refer to
the [`Face` `Intersect` method taking a `Curve` argument](https://www.revitapidocs.com/2020/3513f5e2-a274-4f60-4d8f-78145930a9e3.htm).
I do not understand why you prefer to ask this question here instead of searching for these results yourself.
It took me much longer to write them down than to find them.
I even went ahead and implemented a [line-plane intersection method `LinePlaneIntersection`](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs#L638-L681) for
you in The Building Coder samples.
Here is the code:
```csharp
///
/// Return the 3D intersection point between
/// a line and a plane.
/// https://forums.autodesk.com/t5/revit-api-forum/how-can-we-calculate-the-intersection-between-the-plane-and-the/m-p/9785834
/// https://stackoverflow.com/questions/5666222/3d-line-plane-intersection
/// Determine the point of intersection between
/// a plane defined by a point and a normal vector
/// and a line defined by a point and a direction vector.
/// planePoint - A point on the plane.
/// planeNormal - The normal vector of the plane.
/// linePoint - A point on the line.
/// lineDirection - The direction vector of the line.
/// lineParameter - The intersection distance along the line.
/// Return - The point of intersection between the
/// line and the plane, null if the line is parallel
/// to the plane.
/// summary>
public static XYZ LinePlaneIntersection(
Line line,
Plane plane,
out double lineParameter )
{
XYZ planePoint = plane.Origin;
XYZ planeNormal = plane.Normal;
XYZ linePoint = line.GetEndPoint( 0 );
XYZ lineDirection = (line.GetEndPoint( 1 )
- linePoint).Normalize();
// Is the line parallel to the plane, i.e.,
// perpendicular to the plane normal?
if( IsZero( planeNormal.DotProduct( lineDirection ) ) )
{
lineParameter = double.NaN;
return null;
}
lineParameter = (planeNormal.DotProduct( planePoint )
- planeNormal.DotProduct( linePoint ))
/ planeNormal.DotProduct( lineDirection );
return linePoint + lineParameter \* lineDirection;
}
```
\*\*Response:\*\* Thank you for offering sample code. Actually, I intended to find the solution only with Revit Official API.
Ah well.
By the way, for many other high performance intersection and clipping algorithms, you may want to check
out [Wykobi](https://www.wykobi.com), an
> extremely efficient, robust and simple to use C++ 2D/3D oriented computational geometry library.
![Wykobi segment intersection](img/wykobi_segmentint.png "Wykobi segment intersection")
#### Set Base and Survey Clipped and Unclipped
As we pointed out in the discussion on [survey and project base points](https://thebuildingcoder.typepad.com/blog/2012/11/survey-and-project-base-point.html) in 2012, \*the clipped/unclipped state of the base and survey points could not be set via the API. You could pin them using the Element.Pinned property\*... back then.
Happily and finally, Revit 2021.1 exposed a new property `Clipped` for the base point,
cf. [Clipped state of BasePoint](https://thebuildingcoder.typepad.com/blog/2020/08/revit-20211-sdk-and-whats-new.html#6.3.2)
in [What's New in the Revit 2021.1 API](https://thebuildingcoder.typepad.com/blog/2020/08/revit-20211-sdk-and-whats-new.html).
So, starting from this version, you have the ability to get and set the clipped state for the Survey Point.
For Project Base Point, the property is read-only and will always return false, since the PBP clipped state has been removed.
#### Two German Uni BIM360 Construction Cloud Startups
Moving away from the Revit API to other AEC topics, two innovative BIM360 apps from German Forge developer university startups
are now live, [says](https://twitter.com/ADSK_Construct/status/1311699100312666113)
Phil [@contech101](https://twitter.com/contech101) Mueller, cf.
the [15 new integrations with Autodesk construction cloud](https://constructionblog.autodesk.com/15-integrations-autodesk-construction):
- [Gamma AR](https://construction.autodesk.com/integrations/gamma-ar) – RWTH Aachen
- [4D-Planner](https://construction.autodesk.com/integrations/4d-planner) – TU Berlin
#### AI-Based Face Streaming hits Mainstream
Moving further away from pure AEC related topics,
AI-based face recognition and reconstruction is entering the mainstream through
the [Nvidia Maxine Cloud-AI Video-Streaming Platform](https://developer.nvidia.com/maxine).
It aims to drastically reduce video conferencing bandwidth requirements by transmitting only animated face keypoint data instead of the entire video keyframe information, and reconstructing the animated current presenters face based on some initial video data and the face keypoint data.
Check out the two-and-a-half-minute video on [inventing virtual meetings of tomorrow with Nvidia AI research](https://youtu.be/NqmMnjJ6GEg):

> New AI breakthroughs in NVIDIA Maxine, cloud-native video streaming AI SDK, slash bandwidth use while making it possible to re-animate faces, correct gaze and animate characters for immersive and engaging meetings.