---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: news
optimization_date: '2025-12-11T11:44:13.781747'
original_url: https://thebuildingcoder.typepad.com/blog/0345_manual_regeneration.html
post_number: '0345'
reading_time_minutes: 4
series: general
slug: manual_regeneration
source_file: 0345_manual_regeneration.htm
tags:
- csharp
- elements
- levels
- parameters
- revit-api
- transactions
title: Manual Regeneration Option Danger
word_count: 754
---

### Manual Regeneration Option Danger

Here is a serious issue that several developers have already run into when migrating to the Revit 2011 API, including myself.
It sometimes causes pretty obscure errors which are hard to understand and very easy to resolve, once you understand the source.
Obviously it is best and most efficient to completely avoid running into the problem at all, which I hope this post will help you do.

The issue is the regeneration option, one of the required attributes that must be set on every external command in this version.

For instance, when
[porting The Building Coder samples](http://thebuildingcoder.typepad.com/blog/2010/03/porting-the-building-coder-samples-to-revit-2011.html)
to
Revit 2011,
I globally set the regeneration option to manual, e.g.
```csharp
[Transaction( TransactionMode.Automatic )]
[Regeneration( RegenerationOption.Manual )]
class CmdCollectorPerformance : IExternalCommand
{
  . . .
```

This is a very risky choice to make, since the manual regeneration mode will cause different behaviour than in Revit 2010.

The less risky choice is to set it to automatic, since that will retain the same behaviour experienced in the Revit 2010 API.
If you use manual regeneration mode and your code makes any change to the Revit model and then queries it for data that has been affected by the changes, the returned data will be stale, i.e. out of date.

If in doubt, when migrating moderately complex code to the new version, the safer choice is to always use the automatic mode instead.
One very minor disadvantage of this is that this issues a warning during compilation:

- warning CS0618: 'Autodesk.Revit.Attributes.RegenerationOption.Automatic' is obsolete:
  'Use of RegenerationOpton.Automatic is deprecated and will be removed in a future release.
  Autodesk recommends converting all Revit API code to use the RegenerationOption.Manual option.'

I chose the manual mode for my sample code for several reasons:

- To avoid this compilation warning.- Knowing that almost all of the sample commands are very short and trivial in nature and do not modify the model and query it for data afterwards.- Knowing that this is not production code.

This choice did cause a problem in one of the Revit API introduction labs, Lab2\_0\_CreateLittleHouse, which adds several building elements to the model and in some cases modifies them afterwards.
The area which initially did not work at all and actually threw an exception during execution was the section setting the roof slope:
```csharp
FootPrintRoof roof = createDoc.NewFootPrintRoof(
  profile, levelTop, roofType, out modelCurves );
double slopeAngle = 30 \* LabConstants.DegreesToRadians;
foreach( ModelCurve curve in modelCurves )
{
  roof.set\_DefinesSlope( curve, true );
  roof.set\_SlopeAngle( curve, slopeAngle );
}
```

This code creates a new roof, which returns the automatically generated roof edge curves in the modelCurves array.
It then iterates over them to set the DefinesSlope and SlopeAngle properties on each.
This worked perfectly well in Revit 2010, and also in 2011 with automatic regeneration mode.
With the manual mode, however, it causes the following error:

- Unable to access curves from the roof sketch.

The reason is that the model has been modified by the addition of the model curves, but these changes have not yet propagated far enough through the model for us to already access and modify their properties.
This can easily be resolved once you understand the problem, which simply consists in the addition of a call to doc.Regenerate:
```csharp
FootPrintRoof roof = createDoc.NewFootPrintRoof(
  profile, levelTop, roofType, out modelCurves );
doc.Regenerate();
double slopeAngle = 30 \* LabConstants.DegreesToRadians;
foreach( ModelCurve curve in modelCurves )
{
  roof.set\_DefinesSlope( curve, true );
  roof.set\_SlopeAngle( curve, slopeAngle );
}
```

I discussed another example of using
[manual regeneration mode](http://thebuildingcoder.typepad.com/blog/2010/03/performance-profiling.html#manual_regen) with
Marcelo Quevedo to update the model after a call to SetLocationPoint on a column.
In that case, Marcelo again decided to add appropriate calls to doc.Regenerate to keep the model in synch.

In yet another recent case, Winston Yaw of
[RISA Technologies](http://www.risatech.com) had
a problem accessing the analytical model curve of a newly created slanted column.
Again, the problem was the regeneration option being set to manual.
Once it was set to automatic, everything worked perfectly.
Another issue that Winston encountered that was also caused by the manual regeneration option was setting the vertical projection parameter for a new slab.

In summary, please be aware of the effect of the regeneration option on your code.
If in any doubt whatsoever, when migrating existing code to the Revit 2011 API, your safest bet will be to set it to automatic.