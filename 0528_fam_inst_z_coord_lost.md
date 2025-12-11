---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: documentation
optimization_date: '2025-12-11T11:44:14.109203'
original_url: https://thebuildingcoder.typepad.com/blog/0528_fam_inst_z_coord_lost.html
post_number: 0528
reading_time_minutes: 3
series: general
slug: fam_inst_z_coord_lost
source_file: 0528_fam_inst_z_coord_lost.htm
tags:
- csharp
- elements
- family
- levels
- parameters
- revit-api
title: Family Instance Z Coordinate Lost
word_count: 526
---

### Family Instance Z Coordinate Lost

Here is another contribution by Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de).
He says:

I am working with the placement of family instances on a TopographySurface to represent objects such as arrays of trees, traffic lights, RPC figures etc.
I made use of your discussions on placing
[furniture](http://thebuildingcoder.typepad.com/blog/2010/11/place-furniture-instance.html) and
[detail](http://thebuildingcoder.typepad.com/blog/2010/11/place-detail-instance.html) family
instances and created a Revit add-in providing family instance placement functionality:

![Place family instance on terrain](img/rh_place_instance1.png)
![Place family instance on terrain](img/rh_place_instance2.png)

During the development, we faced some problems similar to those treated in the two blog posts.

There are these steps required to insert a family instance:

1. Place an instance of the family symbol 'fs' using a given XYZ instance 'intersectionPoint':
   ```csharp
     FamilyInstance famInst
       = doc.Create.NewFamilyInstance(
         interSectionPoint, fs,
         StructuralType.UnknownFraming );
   ```- Rotate the family instance.- Set the offset parameter.

When debugging, I found out that the instances were placed correctly in step 1 and their location points had the desired values.

**But if the instances were rotated or moved in step 2, the Location.Z switched back to zero.**

This behaviour only appeared in the English version; in the German one the Revit API acts differently.
In fact, in the German version the FamilyInstance.Location.Z value remains correct even after a rotation.

As a result, we have to check Revit's language and adapt the procedure:

- If it's German, the offset parameter is set correctly, just when placing the family instance.- If it's English, we collect the correct placing data (element, axis and rotation, offset) in a list.
    First, all family instances are placed; after that, we correct their properties using the list like this:

```csharp
  for( int i = 0; i < items.Count; ++i )
  {
    doc.Rotate( items[i], axes[i], rotations[i] );

    doc.Move( items[i], app.Create.NewXYZ(
      0, 0, elevations[i] ) );

    items[i]
      .get\_Parameter( BuiltInParameter
        .INSTANCE\_FREE\_HOST\_OFFSET\_PARAM )
      .Set( elevations[i] );
  }
```

That's our workaround.

In addition, there is another issue:

If the family instance has no visible offset parameter, there is no parameter connected to BuiltInParameter.INSTANCE\_FREE\_HOST\_OFFSET\_PARAM.

In that case it will be impossible to set the offset in both languages.

In the ensuing discussion of this issue, the suggestion came up that this behaviour may well be by design, and a different overload method with a level argument should be used to place the family instance.
Using some overloads, the elevation is fixed and cannot be set manually.
The choice of overload to use
[can be quite tricky](http://thebuildingcoder.typepad.com/blog/2011/01/newfamilyinstance-overloads.html),
as we have seen.

Rudolf responds that just setting the level and fixing the elevation is not the desired result.
Since the language dependent workaround does what I need, there is no time pressure.
I will do some more testing as soon as I find time.
Thank you for your efforts!

Many thanks to Rudolf for this interesting observation, detailed description and rather strange workaround!