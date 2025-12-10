---
post_number: "0486"
title: "Ray Tracing to Place Family Instance"
slug: "ray_trace_place_inst"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'references', 'revit-api', 'views']
source_file: "0486_ray_trace_place_inst.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0486_ray_trace_place_inst.html"
---

### Ray Tracing to Place Family Instance

Jeremiah Farmer of
[Land F/X](http://www.landfx.com) recently
presented his solution for
[placing a site component](http://thebuildingcoder.typepad.com/blog/2010/11/place-detail-instance.html) on
a topo surface.

One important issue in addition to the information presented there is that the family instance insertion point needs to be located on the surface.
One way to find a suitable point is to use the Revit API method
[FindReferencesByDirection](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html) to
shoot a ray through the model to determine a suitable point, i.e. the intersection point between the ray and the surface.
If the desired X and Y position is known, one can use a vertical ray, so that its job is just to determine the appropriate Z coordinate.

Here is Jeremiah's short explanation and sample code demonstrating this:

An update on the Import Site Components, it does seem that a ray trace is necessary to get it correctly anchored at the right Z elevation. So I've added the following:
```vbnet
  Dim surface As TopographySurface \_
    = GetSurface(commandData)

  Dim boundingbox As BoundingBoxXYZ \_
    = surface.BoundingBox(Nothing)

  Dim minZ As Double = boundingbox.Min.Z

  Dim UP As XYZ = XYZ.BasisZ

  Dim view As View = doc.ActiveView

  ' for each point to insert component on surface:

  Dim p0 As XYZ
  ' p1 is our X and Y for that component
  p0 = New XYZ(p1.X, p1.Y, minZ)

  Dim test As ReferenceArray \_
    = doc.FindReferencesByDirection(p0, UP, view)

  If test.Size > 0 Then ' found a surface point
    ' from our desired X,Y, looking Up
    ' from minimum Z to Surface
    p0 = test.Item(0).GlobalPoint
  End If
  ' note if we don't match a surface point,
  ' the call to loadfamilyinstance will fail
```

The GetSurface method is not listed here.
It is a basic helper function similar to ones I've seen previous Building Coder blog posts; it creates a filter, loops through the members, calls TryCast to the element type desired, and returns that element.

Also, it might make sense to add some error handling, since the FindReferenceByDirection method requires a 3D view to be active.
The routine as it now stands will generate an error if a plan view is active.

Note that if no surface point is matched, the subsequent call to insert the family instance will fail.

Many thanks to Jeremiah for his research and sharing this!

By the way, a similar approach is probably also useful for placing furniture family instances onto a floor in the model.