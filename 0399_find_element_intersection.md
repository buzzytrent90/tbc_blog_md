---
post_number: "0399"
title: "Intersection Between Elements"
slug: "find_element_intersection"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'levels', 'references', 'revit-api', 'rooms']
source_file: "0399_find_element_intersection.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0399_find_element_intersection.html"
---

### Intersection Between Elements

One question that regularly comes up is how to determine whether certain building elements intersect.
There are basically two approaches to this, either querying the elements for their pure geometry and making use of pure geometrical intersection methods, or using the higher-level FindReferencesByDirection method, which works on the building elements themselves.
Here is a typical version of this question:

**Question:** I need to find out if any two elements in a project are interfering with each other.
Searching the Revit API help, I found that the Intersect methods on the Autodesk.Revit.Geometry.GeometryObject can be used to calculate intersections.

So it seems that I need to find out the type of geometry for the given building element and then call the relevant intersection method.
Is this correct?
Is there any sample code available demonstrating this?
Is there any API method which accepts two elements and finds out whether they are interfering?

**Answer:** There is no direct API method that calculates the intersection between two Revit building elements.
One possibility is to query them for their geometry elements, and then use the geometry Intersect methods to determine intersection points.

Here is some sample code which shows how to find the intersection points of two geometry lines, excerpted from the Revit SDK sample CreateTruss:
```csharp
  private XYZ GetIntersection(
    Line line1,
    Line line2 )
  {
    IntersectionResultArray results;

    SetComparisonResult result
      = line1.Intersect( line2, out results );

    if( result != SetComparisonResult.Overlap )
      throw new InvalidOperationException(
        "Input lines did not intersect." );

    if( results == null || results.Size != 1 )
      throw new InvalidOperationException(
        "Could not extract line intersection point." );

    IntersectionResult iResult
      = results.get\_Item( 0 );

    return iResult.XYZPoint;
  }
```

A more advanced and complex use of the pure geometrical intersection methods is provided by the AreSolidsCut method defined by the Revit SDK
[RoomsRoofs sample](http://thebuildingcoder.typepad.com/blog/2008/09/roomsroofs-sdk.html).

While there is no direct method to determine whether two building elements intersect, one could implement such a method based on the
[FindReferencesByDirection](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html) method,
as we suggested in the discussion on the
[analytical support tolerance](http://thebuildingcoder.typepad.com/blog/2009/10/analytical-support-tolerance.html).
Several SDK samples demonstrate its use.