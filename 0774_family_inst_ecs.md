---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.565978'
original_url: https://thebuildingcoder.typepad.com/blog/0774_family_inst_ecs.html
post_number: '0774'
reading_time_minutes: 2
series: family
slug: family_inst_ecs
source_file: 0774_family_inst_ecs.htm
tags:
- csharp
- elements
- family
- geometry
- revit-api
title: Family Instance Element Coordinate System
word_count: 462
---

### Family Instance Element Coordinate System

I spent last weekend in Sweden for my 35 year school anniversary, having graduated from the
[German school in Stockholm](http://home.tyskaskolan.se) in
May 1977.

The day after our reunion I visited a friend's summer house on the little lake Kvarnsjön in Huddinge:

![Jerre vid Lasses stuga](file:////j/photo/jeremy/2012/2012-05-28_aaloe/107_jeremy_stuga_2.jpg)

Back to the Revit API, after having taken a second look at
[connector orientation](http://thebuildingcoder.typepad.com/blog/2012/05/connector-orientation.html),
here is another MEP coordinate system topic that can bear updating and coincidentally also has a connection to Scandinavia, since Progman Oy and Olli are Finnish.

I recently presented the discussion with Olli Kattelus of
[MagiCAD](http://www.magicad.com) at
[Progman Oy](http://www.magicad.com/en/content/welcome-house-progman) on
defining an ECS or
[element coordinate system for MEP ducts and pipes](http://thebuildingcoder.typepad.com/blog/2012/03/retrieve-geometry-in-element-coordinate-system.html).

This is required for exporting them to IFC.
Their geometry is specified in ECS there so that the same element definition can be reused consistently for all of its occurrences.

That took care of ducts and pipes, but the issue of family instances was not completely resolved, especially how to handle their potentially 'flipped' geometry.

Now Olli reports an easy solution for that as well, which is actually easier and more obviously
[canonical](http://en.wikipedia.org/wiki/Canonical) than
the duct and pipe one, since a family instance already has a transform that we can base our ECS on:

I managed to solve problems with those flipped objects.

I changed the style how I constructed the transform a little bit.

With fittings, I use the hand/facing orientation & location point of the family instance instead of the connector direction & location:
```csharp
Transform ^tf = Transform::Identity;
// Create correct vectors,
// taking flipping into account
XYZ ^ho = famInstance->HandFlipped
? famInstance->HandOrientation->Negate()
: famInstance->HandOrientation;
XYZ ^fo = famInstance->FacingFlipped
? famInstance->FacingOrientation->Negate()
: famInstance->FacingOrientation;
ho = ho->Normalize();
fo = fo->Normalize();
tf->BasisX = ho;
tf->BasisY = fo;
tf->BasisZ = ho->CrossProduct(fo);
tf->Origin = famInstance->Location->Point;
tf = tf->Inverse;
```

You may be wondering why we need this, since fittings are family instances whose geometry is already defined the way we need it for the IFC export.

However, a fitting may also have an insulation added to it.
The InsulationLiningBase class representing the insulation is inherited from MEPCurve, so we have the same issues for defining an ECS as we did for the duct and curve.

Thus only fittings with insulation need this transform treatment, in order to export their insulation geometry, not the fitting's own one.

Many thanks to Olli for this simple and elegant solution!