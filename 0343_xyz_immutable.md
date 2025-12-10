---
post_number: "0343"
title: "XYZ Immutable"
slug: "xyz_immutable"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'views']
source_file: "0343_xyz_immutable.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0343_xyz_immutable.html"
---

### XYZ Immutable

As mentioned in the
[overview of the Revit 2011 changes](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-is-coming.html),
new classes for XYZ, UV, and ElementId were introduced in the Revit 2011 API.
These ubiquitous objects have been promoted to classes now, some of their methods renamed, and generic .NET collections replace many of the custom Revit collection classes.
The Revit API help What's New section describes these changes in some more detail.

One of the changes which may affect existing code is the immutability of the XYZ coordinate value properties.
The help file suggests that code which previously changed the coordinates of an XYZ instance via setting the values of the X, Y, or Z properties should now construct a new XYZ instance with the desired coordinates instead.

One of the reasons for this change was to better comply with .NET guidelines around exposure of properties.
There were several examples of confusion where API users wrote something like this:
```csharp
  LocationPoint p = famInstance.Location
    as LocationPoint;

  p.Point.X = 0.0;
```
The expectation was to modify the curve's location, but actually it would only change the local copy of the XYZ returned, and not the curve that it came from.
With the immutable XYZ class, this expectation cannot be made, and you are forced to code something like this:
```csharp
  LocationPoint p = famInstance.Location
    as LocationPoint;

  p.Point = new XYZ( 0.0, p.Point.Y, p.Point.Z );
```

Some other examples of changes that this can force you to make are provided by the Revit API introduction lab Lab2\_0\_CreateLittleHouse.
For instance, this line of code was used to offset a point by a certain amount in the Y direction in 2010:
```csharp
  p.Y = p.Y + tagOffset;
```

With the immutable XYZ class in 2011, the Y property can no longer be written to like that.
One possibility is to create a new point instance as suggested above, e.g.
```csharp
  p = new XYZ( p.X, p.Y + tagOffset, p.Z );
```

A slightly shorter alternative is to simply add a vector representing the appropriate offset:
```csharp
  p += tagOffset \* XYZ.BasisY;
```
This also better highlights the fact that we are simply offsetting the point from its original position, and that only the Y coordinate is affected.