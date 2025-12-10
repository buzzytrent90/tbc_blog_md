---
post_number: "0149"
title: "Creating a Slanted Column"
slug: "slanted_column"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'levels', 'parameters', 'revit-api']
source_file: "0149_slanted_column.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0149_slanted_column.html"
---

### Creating a Slanted Column

We already discussed the creation of
[beam](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-beam.html)
and
[column instances](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-column.html)
in previous posts.
In that post on the beam insertion, we explore setting arbitrary start and end points, but the column is left in its default vertical alignment.
Here is a question specifically targeted at the creation of slanted structural columns handled recently by our DevTech AEC workgroup leader Mikako Harada:

**Question:**
I cannot find any help to explain how to create a slanted column through the API.
Please explain the process required, i.e.

- Which NewFamilyInstance method should I use?- Will the parameter be automatically set to Slanted, or will I need to force it after creating the object.- If I need to change an existing vertical column to slanted, how do I do that?- Can I change the location from a point to a line?

**Answer:**
The first place to look for information about the various overloads of NewFamilyInstance and related methods in the Revit API is the developer guide included with the Revit SDK:

- Filename Revit 2010 API Developer Guide.pdf.- Section 10.3.5: Creating FamilyInstance Objects.- Table 28: Options for creating instance with NewFamilyInstance().- Table 29: Options for creating instances with other methods.

The developer guide does not explain how to insert a slanted column, however.
For that, you have two options: the easiest way to go is to use the NewFamilyInstance overload taking the four arguments Curve, FamilySymbol, Level, and StructuralType.
This will allow you to directly create a slanted column.
It will place a column with a SlantedOrVerticalColumnType equal to CT\_EndPoint.

The other approach is explicit, appears more complex, illustrates what is going on behind the scenes, and also demonstrates which parameters and properties need to be modified to create a slanted column:

You can use the same method to create both vertical and slanted columns.
Once the column has been created, you can change its 'Slanted Column Type' property using the built-in parameter SLANTED\_COLUMN\_TYPE\_PARAM.
You can set it to SlantedOrVerticalColumnType CT\_EndPoint, CT\_Angle, or CT\_Vertical.
Once you set the column to a slanted type, the location property becomes a curve, which you can use to set its start and end points.
Here is a snippet of sample code to achieve these steps:

```vbnet
Public Function Execute( \_
    ByVal commandData As ExternalCommandData, \_
    ByRef message As String, \_
    ByVal elements As ElementSet) \_
As IExternalCommand.Result \_
Implements IExternalCommand.Execute

    Dim app As Application = commandData.Application
    Dim doc As Document = app.ActiveDocument

    '' you may want to use your own function
    '' to find these values in your context.
    Dim beamType As FamilySymbol = findSymbol(app)
    Dim topLevel As Level = findLevel(app)

    Dim startPoint As New XYZ(0.0, 0.0, 0.0)
    Dim endPoint As New XYZ(100.0, 100.0, 100.0)

    Dim structuralType As StructuralType = StructuralType.Column

    '' create a coluemn as normal.
    '' at this point, column is vertical.
    Dim strElem As FamilyInstance = doc.Create.NewFamilyInstance( \_
        startPoint, beamType, topLevel, structuralType)

    '' set the property to a slanted column
    Dim param As Parameter = strElem.Parameter( \_
        BuiltInParameter.SLANTED\_COLUMN\_TYPE\_PARAM)

    param.Set(SlantedOrVerticalColumnType.CT\_EndPoint)

    '' after setting to a slanted column,
    '' location should be a curve.
    Dim strElemCurve As LocationCurve = strElem.Location
    '' set the start and end point of a curve.
    If Not (strElemCurve Is Nothing) Then
        Dim line As Line = app.Create.NewLineBound( \_
            startPoint, endPoint)
        strElemCurve.Curve = line
    End If

    Return IExternalCommand.Result.Succeeded

End Function
```

To modify an existing vertical column, you can modify the 'Slanted Column Type' property and then set the two end points in a similar fashion.

Many thanks to Mikako for providing this explanation!

#### Addendum – Cast Built-In Parameter to Integer

In his
[comments](https://thebuildingcoder.typepad.com/blog/2009/06/creating-a-slanted-column.html#comment-4729843697)
[below](https://thebuildingcoder.typepad.com/blog/2009/06/creating-a-slanted-column.html#comment-4732593349),
Matthias Schneider points out that the built-in parameter needs to be converted to `int`:

In Revit 2017 and later versions, this doesn't work any more:

```
  param.Set(SlantedOrVerticalColumnType.CT_EndPoint)
```

Instead, you have to use:

```
  param.Set((int)SlantedOrVerticalColumnType.CT_EndPoint)
```

Many thanks to Matthias this observation!