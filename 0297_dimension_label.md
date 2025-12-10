---
post_number: "0297"
title: "Creating a Dimension Label"
slug: "dimension_label"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'parameters', 'references', 'revit-api', 'views']
source_file: "0297_dimension_label.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0297_dimension_label.html"
---

### Creating a Dimension Label

In a comment on the discussion of the
[family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html),
Nadim asked how to create a label in a family document, and Harry Mattison of Autodesk very friendlily provided some sample source code to generate a new dimension label.
I hope this is what you were looking for.

**Question:** I am trying to use the FamilyItemFactory to create a label inside the family document.
I can see all kinds of creation functions like NewDimension, NewSweep, NewTextNote, but I can't find any function to do NewLabel.
You can do that in the family editor user interface, so it should be possible through the API as well, shouldn't it?
The label might be the most common element used in Annotation Families.

**Answer:** As said, Harry provided a code snippet to create a dimension label in a family document, which I used to implement a new Building Coder sample command CmdNewDimensionLabel.

It creates two model lines which provide the references required to define a dimension and sets up the appropriate reference array.
The reference array is used when calling NewLinearDimension to create a new dimension entity between the two lines.
Finally, a new family parameter named "length" is created and associated with the dimension label.

When I first tried to run this in a new family document based on the annotation family, I was told by Revit that the creation of a new sketch plane is not allowed in that context, so I added a helper method findSketchPlane which returns a sketch plane from the given document with the specified normal vector, if one exists, or else null:
```csharp
static SketchPlane findSketchPlane(
  Document doc,
  XYZ normal )
{
  SketchPlane result = null;

  List<Element> a = new List<Element>();

  int n = doc.get\_Elements(
    typeof( SketchPlane ), a );

  foreach( SketchPlane e in a )
  {
    if( e.Plane.Normal.AlmostEqual( normal ) )
    {
      result = e;
      break;
    }
  }
  return result;
}
```

Unfortunately, in the next step, I noticed that the model lines used in this sample are not allowed in an annotation family either, so I just ran the command in a column family document instead.

Here is the mainline of the CmdNewDimensionLabel external command Execute method which performs the following steps:

- Find or create an appropriate sketch plane to work on, with a normal vector of (0,0,1).- Create two vertical geometry lines.- Create two vertical model curves from the lines.- Collect the two references to the model lines in a reference array.- Create a dimension entity between the two lines.- Create a family parameter named "length" for the dimension label.- Associate the dimension label with the "length" parameter.

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

SketchPlane skplane = findSketchPlane( doc, XYZ.BasisZ );

if( null == skplane )
{
  Plane geometryPlane = app.Create.NewPlane(
    XYZ.BasisZ, XYZ.Zero );

  skplane = doc.FamilyCreate.NewSketchPlane(
    geometryPlane );
}

double length = 1.23;

XYZ start = XYZ.Zero;
XYZ end = app.Create.NewXYZ( 0, length, 0 );

Line line = app.Create.NewLine(
  start, end, true );

ModelCurve modelCurve
  = doc.FamilyCreate.NewModelCurve(
    line, skplane );

ReferenceArray ra = new ReferenceArray();

ra.Append( modelCurve.GeometryCurve.Reference );

start = app.Create.NewXYZ( length, 0, 0 );
end = app.Create.NewXYZ( length, length, 0 );

line = app.Create.NewLine( start, end, true );

modelCurve = doc.FamilyCreate.NewModelCurve(
  line, skplane );

ra.Append( modelCurve.GeometryCurve.Reference );

start = app.Create.NewXYZ( 0, 0.2 \* length, 0 );
end = app.Create.NewXYZ( length, 0.2 \* length, 0 );

line = app.Create.NewLine( start, end, true );

Dimension dim
  = doc.FamilyCreate.NewLinearDimension(
    doc.ActiveView, line, ra );

FamilyParameter familyParam
  = doc.FamilyManager.AddParameter(
    "length",
    BuiltInParameterGroup.PG\_IDENTITY\_DATA,
    ParameterType.Length, false );

dim.Label = familyParam;

return CmdResult.Succeeded;
```

Here is the dimension and the associated dimension label created in a new family document based on the metric column template:

![Dimension with dimension label](img/dimension_label.png)

The length displayed in millimetres, and corresponds to the 1.23 feet specified by the variable 'length', since 1.23 \* 12 \* 25.4 = 374.904.

As noted above, this code will not work unchanged in an annotation family, because you are prohibited from creating model curves there.
To run it in an annotation family, you will have to convert the code to work with
[detail curves](http://thebuildingcoder.typepad.com/blog/2009/09/detail-lines.html)
instead.
You also cannot create a new sketch plane in an annotation family document, which was what prompted me to implement the findSketchPlane helper method.

Here is
[version 1.1.0.61](zip/bc11061.zip)
of the complete Building Coder source code and Visual Studio solution including the new command.

Many thanks to Harry for this solution!