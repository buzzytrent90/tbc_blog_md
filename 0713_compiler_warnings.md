---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.447828'
original_url: https://thebuildingcoder.typepad.com/blog/0713_compiler_warnings.html
post_number: '0713'
reading_time_minutes: 4
series: general
slug: compiler_warnings
source_file: 0713_compiler_warnings.htm
tags:
- csharp
- elements
- filtering
- parameters
- references
- revit-api
- views
- walls
title: Eliminating Compiler Warnings and Deprecated Calls
word_count: 890
---

### Eliminating Compiler Warnings and Deprecated Calls

Migrating Revit add-ins from one version to the next is a regular topic, obviously, since we are generally provided with a new release every year.
Therefore, it is eminently sensible to ensure that an add-in is easy to migrate by avoiding use of all obsolete API functionality.
In some cases, you might also wish to
[support multiple versions](http://thebuildingcoder.typepad.com/blog/2011/11/intermediate-api-update-releases.html).
I provided a simple example of an unusually graceful possibility to handle the differences between the Revit 2011 and 2012
[Idling event handler argument](http://thebuildingcoder.typepad.com/blog/2011/05/cascaded-events-and-renaming-the-active-document.html#2).
Most differences between different versions of the API will probably be slightly more difficult to handle as transparently as this.

Looking back at previous migrations, I discussed The Building Coder samples port from
[Revit 2010 to the Revit 2011 API](http://thebuildingcoder.typepad.com/blog/2010/03/porting-the-building-coder-samples-to-revit-2011.html) and
Joe Ye later
[added more detailed information](http://thebuildingcoder.typepad.com/blog/2010/04/plugin-migration-steps.html) on
that process.

The last time around, when migrating The Building Coder samples from
[Revit 2011 to 2012](http://thebuildingcoder.typepad.com/blog/2011/04/migrating-the-building-coder-samples-to-revit-2012.html),
I provided an even more detailed description of the steps taken.

It still remained pretty quick and dirty, since I performed only very little random testing of the new version, just making sure that the add-in compiled correctly and a few of the numerous commands actually run.

So, until recently, the code was still making use of a number of deprecated API calls which caused compilation warnings.

I removed a few of those when updating the code accessing the
[Materials collection](http://thebuildingcoder.typepad.com/blog/2012/01/materials-collection-and-filtering.html),
where I also provided the latest
[version 2012.0.96.3](http://thebuildingcoder.typepad.com/files/bc_12_96_3-1.zip) of
the solution.

The remaining warnings are caused by use of the Reference class ProximityParameter property and the FindReferencesByDirection method, which was replaced by FindReferencesWithContextByDirection.

Here they are in full – copy to a text editor to see the truncated lines:

```
C:\bc\BuildingCoder\BuildingCoder\CmdNewSpotElevation.cs(69,11): warning CS0618: 'Autodesk.Revit.DB.Document.FindReferencesByDirection(Autodesk.Revit.DB.XYZ, Autodesk.Revit.DB.XYZ, Autodesk.Revit.DB.View3D)' is obsolete: 'This method will be removed, use FindReferencesWithContextByDirection().'
C:\bc\BuildingCoder\BuildingCoder\CmdNewSpotElevation.cs(86,14): warning CS0618: 'Autodesk.Revit.DB.Reference.ProximityParameter' is obsolete: 'Property will be removed. Use ReferenceByContext.ProximityParameter instead (after obtaining ReferenceWithContext from Document.FindReferencesWithContextByDirection())'
C:\bc\BuildingCoder\BuildingCoder\CmdNewSpotElevation.cs(89,21): warning CS0618: 'Autodesk.Revit.DB.Reference.ProximityParameter' is obsolete: 'Property will be removed. Use ReferenceByContext.ProximityParameter instead (after obtaining ReferenceWithContext from Document.FindReferencesWithContextByDirection())'
C:\bc\BuildingCoder\BuildingCoder\CmdDimensionWallsFindRefs.cs(239,14): warning CS0618: 'Autodesk.Revit.DB.Document.FindReferencesByDirection(Autodesk.Revit.DB.XYZ, Autodesk.Revit.DB.XYZ, Autodesk.Revit.DB.View3D)' is obsolete: 'This method will be removed, use FindReferencesWithContextByDirection().'
C:\bc\BuildingCoder\BuildingCoder\CmdDimensionWallsFindRefs.cs(295,17): warning CS0618: 'Autodesk.Revit.DB.Reference.ProximityParameter' is obsolete: 'Property will be removed. Use ReferenceByContext.ProximityParameter instead (after obtaining ReferenceWithContext from Document.FindReferencesWithContextByDirection())'
C:\bc\BuildingCoder\BuildingCoder\CmdDimensionWallsFindRefs.cs(305,19): warning CS0618: 'Autodesk.Revit.DB.Reference.ProximityParameter' is obsolete: 'Property will be removed. Use ReferenceByContext.ProximityParameter instead (after obtaining ReferenceWithContext from Document.FindReferencesWithContextByDirection())'
C:\bc\BuildingCoder\BuildingCoder\CmdDimensionWallsFindRefs.cs(308,34): warning CS0618: 'Autodesk.Revit.DB.Reference.ProximityParameter' is obsolete: 'Property will be removed. Use ReferenceByContext.ProximityParameter instead (after obtaining ReferenceWithContext from Document.FindReferencesWithContextByDirection())'

Compile complete -- 0 errors, 7 warnings
```

Let's tackle those right here and now to future-proof this guy.

It turns out to be really simple.

Here is the original code snippet producing the warnings in the FindTopMostReference method in CmdNewSpotElevation.cs, which determines a reference 'ret' to the topmost face of a given element:
```csharp
  ReferenceArray references
    = doc.FindReferencesByDirection(
      centerOfTopOfBox, viewDirection, view3D );
  foreach( Reference r in references )
  {
    Element re = doc.GetElement( r );

    if( re.Id.IntegerValue == e.Id.IntegerValue
      && r.ProximityParameter < closest )
    {
      ret = r;
      closest = r.ProximityParameter;
    }
  }
```

Instead of FindReferencesByDirection, we now use FindReferencesWithContextByDirection.
It returns instances of the new ReferenceWithContext class instead of the old Reference class.
They are packaged in a generic .NET container, no longer making use of the obsolete custom ReferenceArray collection class.
Finally, the ProximityParameter property has been renamed to just Proximity.

The old code can thus be replaced by the following equivalent new lines:
```csharp
  IList<ReferenceWithContext> references
    = doc.FindReferencesWithContextByDirection(
      centerOfTopOfBox, viewDirection, view3D );
  foreach( ReferenceWithContext r in references )
  {
    Element re = doc.GetElement(
      r.GetReference() );

    if( re.Id.IntegerValue == e.Id.IntegerValue
      && r.Proximity < closest )
    {
      ret = r.GetReference();
      closest = r.Proximity;
    }
  }
```

I made similar changes in the other module using this same ray casting functionality, CmdDimensionWallsFindRefs.cs, and now the entire solution compiles with zero warnings.

Here is
[version 2012.0.96.4](zip/bc_12_96_4.zip) of
The Building Coder samples all deprecated API calls eliminated.

Most of the obsolete lines are marked with comments saying '// 2011', and their replacements are marked '// 2012'.
If you wish to analyse the exact differences, you can simply compare the code of this version with the
[version 2012.0.96.3](http://thebuildingcoder.typepad.com/files/bc_12_96_3-1.zip) mentioned above.