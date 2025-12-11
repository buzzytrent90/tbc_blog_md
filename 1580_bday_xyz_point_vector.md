---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.7
content_type: qa
optimization_date: '2025-12-11T11:44:16.332363'
original_url: https://thebuildingcoder.typepad.com/blog/1580_bday_xyz_point_vector.html
post_number: '1580'
reading_time_minutes: 7
series: geometry
slug: bday_xyz_point_vector
source_file: 1580_bday_xyz_point_vector.md
tags:
- elements
- geometry
- references
- revit-api
- sheets
- views
title: Bday Xyz Point Vector
word_count: 1317
---

### Birthday Post on the XYZ Class
Today is The Building Coder's ninth birthday.
The first [welcome](http://thebuildingcoder.typepad.com/blog/2008/08/welcome.html) post was published August 22, 2008.
We'll celebrate by discussing the pretty fundamental issue of `XYZ` points versus vectors, and how to distinguish different points:
- [`XYZ` point versus vector](#2)
- [How to distinguish `XYZ` points](#3)
![Vector from A to B](img/Vector_from_A_to_B.png)
#### XYZ Point versus Vector
This question was raised in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`XYZ` question](https://forums.autodesk.com/t5/revit-api-forum/xyz-question/m-p/6460982):
\*\*Question:\*\* I find the `XYZ` class very confusing because it can be a point or a vector.
Does anybody know any good guide on how to use them? Or some documentation.
Also, I have a specific question: how does one extract start and end points of a XYZ vector?
\*\*Answer:\*\* The documentation is right there where it belongs, in the Revit API help file `RevitAPI.chm`
[section on the `XYZ` class](http://www.revitapidocs.com/2017/c2fd995c-95c0-58fb-f5de-f3246cbc5600.htm).
You are perfectly right, the XYZ class can represent either a point or a vector, depending on how you use it.
If you have a `XYZ` object and run an angle method, you're in fact handling a vector information... but if you just get a coordinate, then a point.
In some cases, a vector doesn't have a start or end point, for instance, if it is used to represent a pure direction, not a line. If you need a line, use the Curve/Line object.
[Bobby Jones](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/53074) suggested rolling your own `Point` and `Vector` wrappers around it, making your code much easier to maintain.
The `XYZ` class defines methods that are specific to the concept of both points and vectors. Sometimes, it is difficult to know which concept it is dealing with at any given time.
Another benefit of providing your own point and vector wrappers is that you can limit their interface to only methods that make sense.
The following code compiles, but none of it makes any sense:
```csharp
void PointAndVectorExample( Line revitLine )
{
var o = revitLine.Origin;
var whatIstheLengthOfaPoint = o.GetLength();
var howIsaPointaUnitLength = o.IsUnitLength();
var lineDirection = revitLine.Direction;
var whatDoesThisRepresent = lineDirection.CrossProduct( o );
var thisDoesntMakeSense = lineDirection.DistanceTo( o );
}
```
Here are some PARTIAL `Point3D` and `Vector3D` classes to give you the idea, and some helper extensions to make them easier to use. You can take these and wrap the appropriate `XYZ` methods for each as well as define additional methods of your own. And even implement additional interfaces, such as `IEquatable`.
```csharp
public class Point3D
{
public Point3D( XYZ revitXyz )
{
XYZ = revitXyz;
}
public Point3D() : this( XYZ.Zero )
{ }
public Point3D( double x, double y, double z )
: this( new XYZ( x, y, z ) )
{ }
public XYZ XYZ { get; private set; }
public double X => XYZ.X;
public double Y => XYZ.Y;
public double Z => XYZ.Z;
public double DistanceTo( Point3D source )
{
return XYZ.DistanceTo( source.XYZ );
}
public Point3D Add( Vector3D source )
{
return new Point3D( XYZ.Add( source.XYZ ) );
}
public static Point3D operator +(
Point3D point,
Vector3D vector )
{
return point.Add( vector );
}
public override string ToString()
{
return XYZ.ToString();
}
public static Point3D Zero
=> new Point3D( XYZ.Zero );
}
public class Vector3D
{
public Vector3D( XYZ revitXyz )
{
XYZ = revitXyz;
}
public Vector3D() : this( XYZ.Zero )
{ }
public Vector3D( double x, double y, double z )
: this( new XYZ( x, y, z ) )
{ }
public XYZ XYZ { get; private set; }
public double X => XYZ.X;
public double Y => XYZ.Y;
public double Z => XYZ.Z;
public Vector3D CrossProduct( Vector3D source )
{
return new Vector3D( XYZ.CrossProduct( source.XYZ ) );
}
public double GetLength()
{
return XYZ.GetLength();
}
public override string ToString()
{
return XYZ.ToString();
}
public static Vector3D BasisX => new Vector3D(
XYZ.BasisX );
public static Vector3D BasisY => new Vector3D(
XYZ.BasisY );
public static Vector3D BasisZ => new Vector3D(
XYZ.BasisZ );
}
public static class XYZExtensions
{
public static Point3D ToPoint3D( this XYZ revitXyz )
{
return new Point3D( revitXyz );
}
public static Vector3D ToVector3D( this XYZ revitXyz )
{
return new Vector3D( revitXyz );
}
}
public static class LineExtensions
{
public static Vector3D Direction(
this Line revitLine )
{
return new Vector3D( revitLine.Direction );
}
public static Point3D Origin(
this Line revitLine )
{
return new Point3D( revitLine.Origin );
}
}
```
Many thanks to Bobby for sharing this!
#### How to Distinguish XYZ Points
Another [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
that Bobby also helped out with deals
with [Distinct `XYZ`](https://forums.autodesk.com/t5/revit-api-forum/xyz-question/m-p/6460982):
\*\*Question:\*\* I have a list of `XYZ` points obtained from MEP connectors.
How can I clean it of duplicates, i.e., eliminate points with identical coordinates?
I am trying to do it like this:
```csharp
var distinctElementConnectors
= MyMepUtils.GetALLConnectors( elements )
.Where( c => c.IsConnected )
.Distinct( c => c.Origin )
.ToHashSet();
```
The call to `Distinct(c => c.Origin)` doesn't work, because `Distinct` doesn't know how to compare XYZs (or does it?).
\*\*Answer:\*\* You are absolutely correct:
The .NET API does not have any built-in mechanism to compare the Revit API `XYZ` objects.
However, it is easy to implement, and I have done so several times in different discussion published by The Building Coder.
Here are the first and last mentions so far:
- [Nested instance geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html)
- [Fuzzy comparison](http://thebuildingcoder.typepad.com/blog/2017/06/sensors-bim-ai-revitlookup-and-fuzzy-comparison.html#4)
Furthermore, you can take a look
at [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) class
`XyzEqualityComparer`, defined there in three different modules.
Another direction to go, again suggested by Bobby, assuming you implemented your own `Point3D` and `Vector3D` wrapper classes, is to have those classes implement the `IEquatable` interface.
Here's a shell of the `Point3D` class showing an implementation:
```csharp
public static class DoubleExtensions
{
private const double Tolerance = 1.0e-10;
public static bool IsAlmostEqualTo(
this double double1,
double double2 )
{
var isAlmostEqual =
Math.Abs( double1 - double2 )
<= Tolerance;
return isAlmostEqual;
}
}
public class Point3D : IEquatable
{
public Point3D( XYZ revitXyz )
{
XYZ = revitXyz;
}
public XYZ XYZ { get; }
public double X => XYZ.X;
public double Y => XYZ.Y;
public double Z => XYZ.Z;
public bool Equals( Point3D other )
{
if( ReferenceEquals( other, null ) ) return false;
if( ReferenceEquals( this, other ) ) return true;
return X.IsAlmostEqualTo( other.X ) &&
Y.IsAlmostEqualTo( other.Y ) &&
Z.IsAlmostEqualTo( other.Z );
}
public override bool Equals( object obj )
{
if( ReferenceEquals( null, obj ) ) return false;
if( ReferenceEquals( this, obj ) ) return true;
if( obj.GetType() != GetType() ) return false;
return Equals( (Point3D) obj );
}
public override int GetHashCode()
{
return Tuple.Create( Math.Round( X, 10 ),
Math.Round( Y, 10 ),
Math.Round( Z, 10 ) ).
GetHashCode();
}
}
```
Storing `Point3D` instances in a hashset will give you your distinct set of points.
```csharp
var distinctElementConnectors
= MyMepUtils.GetALLConnectors( elements )
.Where( c => c.IsConnected )
.Select( c => c.Origin.ToPoint3D() )
.ToHashSet();
```
Yet another solution to address this directly is to define a comparer class for native Revit API `Connector` objects, such as the `ConnectorXyzComparer` one provided in
at [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) [Util.cs module](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs#L1439-L1465):
```csharp
///
/// Compare Connector objects based on their location point.
/// summary>
public class ConnectorXyzComparer : IEqualityComparer
{
public bool Equals( Connector x, Connector y )
{
return null != x
&& null != y
&& IsEqual( x.Origin, y.Origin );
}
public int GetHashCode( Connector x )
{
return HashString( x.Origin ).GetHashCode();
}
}
///
/// Get distinct connectors from a set of MEP elements.
/// summary>
public static HashSet GetDistinctConnectors(
List cons )
{
return cons.Distinct( new ConnectorXyzComparer() )
.ToHashSet();
}
```
I implemented that in The Building Coder samples [release 2018.0.134.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.134.1).
Here is the [diff to the preceding release](https://github.com/jeremytammik/the_building_coder_samples/compare/2018.0.134.0...2018.0.134.1).
Thanks again to Bobby, and Happy Birthday coding, everybody!