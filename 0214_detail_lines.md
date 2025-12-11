---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:13.563424'
original_url: https://thebuildingcoder.typepad.com/blog/0214_detail_lines.html
post_number: '0214'
reading_time_minutes: 3
series: general
slug: detail_lines
source_file: 0214_detail_lines.htm
tags:
- csharp
- elements
- geometry
- parameters
- revit-api
- views
title: Detail Lines
word_count: 626
---

### Detail Lines

We previously discussed various aspects of model lines, such as
[model line creation](http://thebuildingcoder.typepad.com/blog/2008/11/model-line-creation.html),
the
[model line sketch plane](http://thebuildingcoder.typepad.com/blog/2009/05/model-line-sketch-plane.html),
and the
[minimal length of a model line](http://thebuildingcoder.typepad.com/blog/2009/07/think-big-in-revit.html).
Here is a question on the related topic of detail lines handled by my colleague Saikat Bhattacharya:

**Question:**
How can I create a detail line on a drafting view?
I tried to use the following statement:

```
doc.Create.NewDetailCurve( View, curve, y );
```

It is unclear to me what the "curve" and the "sketchPlane" stand for and how to use them.
Please could you provide me with a small sample.

**Answer:**
Detail curves are created for drawings such as a detail or drafting view.
They are annotation elements and are visible only in the view in which they are drawn.
They can currently only be created in plan views.

Except for this, detail curves are similar to model curves.
ModelLine and DetailLine both belong to the Lines category.
Thus, for a sample on creating detail curves, you can refer to and slightly adapt the Revit SDK sample ModelLines,
which shows how to work with the sketch plane and geometry curve to create a new model line.

For more details on sketch planes and model curves, you can also refer to the developer guide "Revit 2010 API Developer Guide.pdf", especially chapter 14, Annotation Elements, and section 14.2, Detail Curve.
Another helpful section is 15.3.1 on Creating a ModelCurve and the Code Region 15-2: Creating a new model curve, which illustrates how to create new geometry curves, a sketch plane, and model curves, a ModelLine and a ModelArc.
You can use the same approach to create detail curves as well.

I implemented a new external command CmdDetailCurves in The Building Coder sample application by slightly modifying the sample code provided in the developer guide.
Here is the code of the new command's Execute method:

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
  // created geometry line and sketch plane
  DetailLine line = doc.Create.NewDetailCurve(
    view, geomLine ) as DetailLine;

  // Create a DetailArc element using the
  // created geometry arc and sketch plane
  DetailArc arc = doc.Create.NewDetailCurve(
    view, geomArc ) as DetailArc;

  return CmdResult.Succeeded;
```

As you see, the curve parameter above can be any geometric curve entity like Line, Arc, Ellipse or NurbSpline.
All of these types are derived from the Curve class and can be used as parameters to create a detail curve.

Here are the resulting detail curves after running this command in a new project document:
![Detail curves](img/detail_curves.png)

Here is
[version 1.1.0.46](zip/bc11046.zip)
of the complete Building Coder sample source code and Visual Studio solution including the new command.

Thank you Saikat for this helpful explanation!

**Addendum:** Please make sure you do not overlook the
[update to this post](http://thebuildingcoder.typepad.com/blog/2010/05/detail-curve-must-indeed-lie-in-plane.html).