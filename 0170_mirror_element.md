---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.480404'
original_url: https://thebuildingcoder.typepad.com/blog/0170_mirror_element.html
post_number: '0170'
reading_time_minutes: 2
series: elements
slug: mirror_element
source_file: 0170_mirror_element.htm
tags:
- csharp
- elements
- family
- geometry
- levels
- references
- revit-api
- selection
title: Mirror an Element
word_count: 331
---

### Mirror an Element

We already discussed
[transformation of elements](http://thebuildingcoder.typepad.com/blog/2009/05/transform-an-element.html) using Move and Rotate.
Here is a question regarding the related method Mirror that also deserves attention:

**Question:**
I am trying to mirror a family instance element using the X axis as the mirror line.
I am creating a new line using the NewLine method and then supplying a reference to that to the Document.Mirror method, but this is causing Revit to crash.
The Reference property of my line is always null.
I am doing it like this because I saw an example showing how to mirror a column element through its analytical model curve reference.
How can I successfully mirror my family instance element if I have no such reference?

**Answer:**
What you experience is expected.
In your code, you are creating an abstract geometrical line at the application level.
The resulting object is an abstract geometry entity and not a document element, and therefore no reference is available for this entity.
In this case, you should not use the overload of the Mirror method that takes a Reference argument.
You can simply use the one taking a Geometry.Line argument instead and use the line itself directly.
If you really wanted to use a reference, you could also create a model line in the document and access its reference using the Line.GeometryCurve.Reference property.
Normally, one would prefer not to create model elements if it is possible to use only in memory objects.
Here is the complete code of a minimal but complete external command Execute method which mirrors the currently selected elements around the X axis:

```csharp
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  ElementSet els = doc.Selection.Elements;

  Line line = app.Create.NewLine(
    XYZ.Zero, XYZ.BasisX, true );

  doc.Mirror( els, line );

  return CmdResult.Succeeded;
}
```

Many thanks to Saikat Bhattacharya for handling this case!