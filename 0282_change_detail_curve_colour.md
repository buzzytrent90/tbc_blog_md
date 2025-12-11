---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:13.672825'
original_url: https://thebuildingcoder.typepad.com/blog/0282_change_detail_curve_colour.html
post_number: 0282
reading_time_minutes: 2
series: geometry
slug: change_detail_curve_colour
source_file: 0282_change_detail_curve_colour.htm
tags:
- csharp
- elements
- geometry
- revit-api
- views
title: Model and Detail Curve Colour
word_count: 483
---

### Model and Detail Curve Colour

We have had several questions on how to change the colour of model or detail curves.
Here is an answer to this question from a recent case handled by Joe:

**Question:** I am drawing a detail arc using the Revit API DetailArc class and
trying to change its properties like colour, graphic style etc.
How can this be achieved?

**Answer:** You can change the detail arc line colour by changing the colour of the detail curve category's line style.

Here is The Building Coder sample command
[CmdDetailCurves](http://thebuildingcoder.typepad.com/blog/2009/09/detail-lines.html)
demonstrating the creation of a few detail curves and enhanced to modify the line colour of their graphics style:
```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;
View view = doc.ActiveView;

// Create a geometry line

XYZ startPoint = new XYZ( 0, 0, 0 );
XYZ endPoint = new XYZ( 10, 10, 0 );

Line geomLine = app.Create.NewLine(
  startPoint, endPoint, true );

// Create a geometry arc

XYZ end0 = new XYZ( 1, 0, 0 );
XYZ end1 = new XYZ( 10, 10, 10 );
XYZ pointOnCurve = new XYZ( 10, 0, 0 );

Arc geomArc = app.Create.NewArc(
  end0, end1, pointOnCurve );

// Create a geometry plane

XYZ origin = new XYZ( 0, 0, 0 );
XYZ normal = new XYZ( 1, 1, 0 );

Plane geomPlane = app.Create.NewPlane(
  normal, origin );

// Create a sketch plane in current document

SketchPlane sketch = doc.Create.NewSketchPlane(
  geomPlane );

// Create a DetailLine element using the
// newly created geometry line and sketch plane

DetailLine line = doc.Create.NewDetailCurve(
  view, geomLine ) as DetailLine;

// Create a DetailArc element using the
// newly created geometry arc and sketch plane

DetailArc arc = doc.Create.NewDetailCurve(
  view, geomArc ) as DetailArc;

// Change detail curve colour.
// Initially, this only affects the newly
// created curves. However, when the view
// is refreshed, all detail curves will
// be updated.

GraphicsStyle gs = arc.LineStyle as GraphicsStyle;

gs.GraphicsStyleCategory.LineColor
  = new Color( 250, 10, 10 );

return CmdResult.Succeeded;
```

Note that you need to ensure that detail curves are visible in the view that you are working in in order to see the changes taking effect.

Also note that the newly defined colour will be applied to all detail curves after the view is refreshed or closed and reopened.
It is not possible to change one single detail line's colour individually in the long run.

Here are a couple of detail curves drawn in the default drawing before running this command:
![Default detail curves](img/detail_curves_1.png)

Here are the new detail curves added by the command with their modified line colour:
![Detail curves with modified settings](img/detail_curves_2.png)

When the view is refreshed or closed and reopened, all detail lines will adopt the new colour.

Here is
[version 1.1.0.58](zip/bc11058.zip)
of the complete Building Coder sample source code and Visual Studio solution including the updated version of this command.

Many thanks to Joe for handling this case!