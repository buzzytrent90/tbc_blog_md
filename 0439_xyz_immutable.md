---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: qa
optimization_date: '2025-12-11T11:44:13.948098'
original_url: https://thebuildingcoder.typepad.com/blog/0439_xyz_immutable.html
post_number: 0439
reading_time_minutes: 2
series: geometry
slug: xyz_immutable
source_file: 0439_xyz_immutable.htm
tags:
- csharp
- elements
- parameters
- revit-api
- views
- walls
- geometry
title: Immutable PointLoad Force
word_count: 492
---

### Immutable PointLoad Force

Today we crossed the
[Kattegat](http://en.wikipedia.org/wiki/Kattegat) from
[Læsø](http://en.wikipedia.org/wiki/L%C3%A6s%C3%B8) in Denmark to
[Rörö](http://en.wikipedia.org/wiki/R%C3%B6r%C3%B6) in Sweden,
north of Göteborg, so we have finally arrived in the archipelago on the west coast of Sweden.
As said, you can follow our exact route on the
[Pantagruel tracker](http://share.findmespot.com/shared/faces/viewspots.jsp?glId=0fQEu0hRkYX5FQpeMDEj8hms84EuXOsRl).

We spent more than 48 hours at sea when going from Norway to Denmark, with some very beautiful moments, among others the night watches with more stars than you can imagine.
I saw four shooting stars the first night.
And fluorescent
[plankton](http://en.wikipedia.org/wiki/Plankton) in our wake.

Today we had quite strong winds and made more than ten knots several times, which was nice.

Looking at the Revit API again, we already discussed the
[immutability of the XYZ class](http://thebuildingcoder.typepad.com/blog/2010/04/xyz-immutable.html) in
the Revit 2011 API.
Here is another aspect of that modification, from a case handled by
[Augusto Gonçalves](http://thebuildingcoder.typepad.com/blog/2010/08/edit-wall-length.html) and Scott Conover:

**Question:** Using the Revit 2010 API, I was able to modify the X, Y and Z components of the PointLoad Force and Moment properties, e.g.
```csharp
PointLoad.Force.X
PointLoad.Force.Y
PointLoad.Force.Z
PointLoad.Moment.X
PointLoad.Moment.Y
PointLoad.Moment.Z
```
In the Revit 2011 API, these values are read-only.
How can I modify this data in Revit 2011?

**Answer:** Actually, even in the Revit 2010 API, changing the value of the X, Y and Z components of the PointLoad Force and Moment properties never had any effect on the underlying Revit model.
The XYZ data returned by these properties is and always has been a copy of the underlying data, so changing it does not affect and never has affected the actual force or moment value inside the point load.

One of the main reasons why the
[XYZ class is now immutable](http://thebuildingcoder.typepad.com/blog/2010/04/xyz-immutable.html) is
to ensure that this mistake cannot be assumed to work anymore.

If the PointLoad Force and Moment properties were not read-only properties, you would be able to change their values by assigning a new XYZ value to the PointLoad element property, e.g.
```csharp
pointLoad.Force = new XYZ( 1.0, 2.0, 3.0 );
```

Unfortunately they are read-only, so this obvious approach cannot be used in this case.
It would however work for other, similar, examples.

In this case, the modification can be achieved via the parameter associated with the appropriate built-in parameter enum, e.g. BuiltInParameter.LOAD\_FORCE\_FY:
```csharp
Parameter p = pointLoad.get\_Parameter(
BuiltInParameter.LOAD\_FORCE\_FY );
p.Set( 10.00 );
```

Many thanks to Scott and Augusto for the explanation!