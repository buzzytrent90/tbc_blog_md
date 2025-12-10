---
post_number: "0875"
title: "GetInstanceGeometry Overhead and Exceptions"
slug: "inst_geom_overhead"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'references', 'revit-api', 'views']
source_file: "0875_inst_geom_overhead.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0875_inst_geom_overhead.html"
---

### GetInstanceGeometry Overhead and Exceptions

I made it back safe and sound to Switzerland from the developer conferences, and I note that this special pivotal day – the
[winter solstice](http://en.wikipedia.org/wiki/Winter_solstice) –
seems to be progressing normally and we all survived in spite of the end of an era in the
[Maya calendar](http://en.wikipedia.org/wiki/Maya_calendar).

Thus we can continue growing our understanding of the Revit API, as well as attending to other important business here on our lovely planet.

The GeometryInstance GetInstanceGeometry method returns the geometry of a family symbol.
More precisely, according to the Revit API help file RevitAPI.chm documentation, it computes the geometric representation of the instance.

Computes really means computes, and incurs some significant overhead, as emphasised by the help file remarks, which state:

The geometry will be in the coordinate system of the model that owns this instance.
The context of the instance object (such as effective material) will be applied to the symbol.
Note that this method involves extensive parsing or Revit's data structures, so try to minimize calls if performance is critical.
Geometry will be parsed with the same options as those used when this object was retrieved.
This method returns a copy of the Revit geometry.
It is suitable for use in a tool which extracts geometry to another format or carries out a geometric analysis; however, because it returns a copy, the references found in the geometry objects contained in this element are not suitable for creating new Revit elements referencing the original element (for example, dimensioning).
Only the geometry returned by GetSymbolGeometry with no transform can be used for that purpose.

This is also covered by the Developer Guide section on
[geometry instances](http://wikihelp.autodesk.com/Revit/enu/2013/Help/00006-API_Developer%27s_Guide/0074-Revit_Ge74/0108-Geometry108/0110-Geometry110/GeometryInstances).

Now a poor hapless (or not so hapless, in fact) developer ran into an example where the invalid references returned by this method cause a significant problem, providing a welcome opportunity to highlight the importance of avoiding use of this particular method for that kind of use:

**Question:** I am using the analysis visualisation framework AVF to paint some transient graphics a given face.

When I use the following line to apply the AVF to the face, the behaviour is correct:

```
  int idx = sfm.AddSpatialFieldPrimitive(
    face, Transform.Identity );
```

When this call is used instead, however, Revit throws an exception saying that the reference must point to either a Face or a Curve visible in the view:

```
  int idx = sfm.AddSpatialFieldPrimitive(
    face.Reference );
```

This message is incorrect, because the Face **is** visible in this view.

I really need the face reference call to work so that the AddSpatialFieldPrimitive method overload taking a reference and SpatialFieldPrimitiveHideMode can be used, i.e.

```
  AddSpatialFieldPrimitive(
    Reference,
    SpatialFieldPrimitiveHideMode )
```

**Answer:** The behaviour you observe is caused by your use of the GeometryInstance GetInstanceGeometry method, and then asking AVF to pull references from that geometry.

As stated in the description quoted above, these references are invalid.

If you simply change your macro to use GetSymbolGeometry instead, the AVF results will be correctly painted on the faces.

Some other operations besides AVF that also need these references to be correctly obtained from GetSymbolGeometry include dimensioning, face based family creation, and sketch plane creation.

GetInstanceGeometry is a shortcut to get 'in-place' geometry for operations like export to another format, Boolean operations with another solid, or other geometry tools that don't need the connection back to the original element.

Many thanks to Scott Conover for clarifying this!