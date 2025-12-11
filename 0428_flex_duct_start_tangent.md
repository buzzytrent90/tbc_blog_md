---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: code_example
optimization_date: '2025-12-11T11:44:13.930048'
original_url: https://thebuildingcoder.typepad.com/blog/0428_flex_duct_start_tangent.html
post_number: 0428
reading_time_minutes: 3
series: mep
slug: flex_duct_start_tangent
source_file: 0428_flex_duct_start_tangent.htm
tags:
- csharp
- family
- parameters
- revit-api
- mep
title: Flex Duct Start Tangent
word_count: 535
---

### Flex Duct Start Tangent

Still zipping around in Pattaya with my brother Marcus, and still blogging a bit now and then as well ...

Here is a suggestion for a workaround to specify the start and end tangents of a flex duct or pipe that evolved through a recent
[exchange](http://thebuildingcoder.typepad.com/blog/2010/08/structural-dynamic-model-update-sample.html?cid=6a00e553e168978833013485f297dc970c#comment-6a00e553e168978833013485f297dc970c)
[of](http://thebuildingcoder.typepad.com/blog/2010/08/structural-dynamic-model-update-sample.html?cid=6a00e553e1689788330133f2dfa073970b#comment-6a00e553e1689788330133f2dfa073970b)
[comments](http://thebuildingcoder.typepad.com/blog/2010/08/structural-dynamic-model-update-sample.html?cid=6a00e553e1689788330134860dce69970c#comment-6a00e553e1689788330134860dce69970c) with Zhu XiaoMeng:

**Question:** In the Revit application UI, I can create a FlexDuct properly with the start tangent of the Hermite curve in a reasonable direction.
But when I create a flex duct in the API, the start tangent is always horizontal.
How can I do the same thing in API as in the UI?

The initial code is as follows:
```csharp
  FlexDuct flexDuct = doc.Create.NewFlexDuct(
    points, flexducttype );
```

**Answer:** One thing to note regarding this is that it does not seem to be related to the
[Hermite spline issue](http://thebuildingcoder.typepad.com/blog/2010/07/debugging-in-visual-studio-2010-express.html?cid=6a00e553e168978833013485ce109c970c#comment-6a00e553e168978833013485ce109c970c) that we recently discussed.

Here is a suggestion for a possible workaround:

1. Place a fitting with a connector temporarily where the direction of the connector matches the desired tangent.- Place the flex duct or pipe and attach it to the connector.- Delete the fitting.

**Response:** I did as what you say, but I get confused with the second step.
I placed a fitting with two side connectors in the right place, and then placed the flex duct in the right position, but they did not join.
So the tangent did not change.
So the point is: how can I get them joined?

Here is my current code:
```csharp
  Family fittingFamily
    = lstElem.Where( e => e.Name == "M\_xyz" )
    .Select( e => e as Family )
    .FirstOrDefault();

  FamilySymbol fittingType = fittingFamily
    .Symbols
    .ToList()
    .First();

  const double fitLen = 1.5;

  var vectDown
    = ( samplePoints[1] - samplePoints[0] )
      .Normalize();

  var fittPos = ptA - vectDown \* fitLen;

  // vectAb is the vector pointing from
  // ptA(samplePoints[First]) to
  // ptB(samplePoints[Last])

  //  remove the z values (as the
  // flexDuct first tangent do)

  var vectAbHoriz = new XYZ( vectAb.X, vectAb.Y, 0 );

  FamilyInstance fittingInstance
    = \_doc.Create.NewFamilyInstance(
      fittPos, fittingType,
      StructuralType.NonStructural );

  Line axis2 = \_app.Create.NewLine(
    fittPos, vectDown.CrossProduct( XYZ.BasisZ ),
    false );

  // rotate the fitting to the desired angle

  bool success = \_doc.Rotate( fittingInstance,
    axis2, -vectDown.AngleTo( vectAbHoriz ) );

  SortedList valuePairs = new SortedList();

  valuePairs["xyz 01"] = fitLen;

  ChangeParametersValue( fittingInstance, valuePairs );

  FlexDuct flexDuct2 = CreateDuct(
    samplePoints, flexDuctType );
```

**Answer:** You probably have to explicitly connect them with each other as demonstrated in the AutoRoute and AvoidObstruction SDK samples.

**Response:** Thank you, it's solved!

Here is my way in case some one else encounters the same problem:

Find the right connector of the fitting as well as the right connector of flexDuct, and call the connector ConnectTo method to join them.

Before deleting the temporary fitting, call the document Regenerate method, otherwise the tangent will not change!

Very many thanks to Zhu XiaoMeng for this discussion and sharing the successful result!