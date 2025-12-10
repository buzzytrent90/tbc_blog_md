---
post_number: "0636"
title: "Retrieving Lines Within a Family Instance"
slug: "lines_in_inst"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'levels', 'references', 'revit-api', 'selection', 'views', 'walls']
source_file: "0636_lines_in_inst.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0636_lines_in_inst.html"
---

### Retrieving Lines Within a Family Instance

Here is a basic query on how to obtain geometrical objects from a family instance answered by my colleague Saikat Bhattacharya.
We have looked at more complex geometry retrieval in the past, so let's look at this simple case now:

**Question:** I am examining an instance of a family created from a Generic Model containing a parametric sketch.
I loaded such a family into my project and inserted an instance of it.
The idea is to use the parametric sketch to create different walls in the project.
If I snoop the resulting FamilyInstance inserted from the loaded family, I can see the lines in the collection FamilyInstance > FamilySymbol > Objects > Lines.
I cannot find the proper path to access this data through my own code, however.
What can I do to access the sketch lines programmatically?
I tried the following:

- myFamilyInstance.Symbol.GetGeometryObjectFromReference(Reference ref)

  Here, I don't understand the purpose of the Reference element.- myFamilyInstance.Symbol.get\_Geometry(Options opt)

    Here, I don't know what I should pass in as option.

Finally, I'm not sure I'm doing the right thing.
What I want to do in the end is to pass in the lines contained in the FamilyInstance element to wall creation method to define new walls based on the generic model geometry.

**Answer:** You were close.
Here is a code snippet in which I retrieve the lines you are looking for and simply print out their length:
```vbnet
Public Function Execute(
  ByVal commandData As ExternalCommandData,
    ByRef message As String,
    ByVal elements As ElementSet) \_
  As Result Implements IExternalCommand.Execute

  Dim uiapp As Application \_
    = commandData.Application

  Dim activeDoc As Document \_
    = uiapp.ActiveUIDocument.Document

  Dim opts As Options \_
    = activeDoc.Application.Create.NewGeometryOptions()

  Dim elem As Element

  For Each elem In activeDoc.Selection.Elements

    Dim famInst As FamilyInstance \_
      = CType(elem, FamilyInstance)

    Dim geoEle As GeometryElement \_
      = famInst.Geometry(opts)

    Dim geoInstance As GeometryInstance

    For Each geoInstance In geoEle.Objects
      Dim symbIns As GeometryObject

      For Each symbIns In geoInstance.
        SymbolGeometry.Objects

        If TypeOf (symbIns) Is Line Then
          Dim line As Line = symbIns
          TaskDialog.Show(
            "Line Length",
            line.ApproximateLength.ToString())
        End If
      Next
    Next
  Next

  Return Result.Succeeded

End Function
```

When querying an element for its geometry, you do indeed need to pass in an instance of the Options class, which is described in greater detail in the developer guide document "Revit 2012 API Developer Guide.pdf".

The geometry options include the Options.View property, which specifies the
[view used for geometry extraction](http://thebuildingcoder.typepad.com/blog/2011/08/section-view-geometry.html).
If a view-specific version of an element exists, it will be extracted in the retrieval of geometry.
Also, the detail level of the geometry will be taken from the view's detail level.

If you prefer the code in C# rather than VB, you can convert it using one of the many free online translators such as
[developerFusion](http://www.developerfusion.com/tools/convert/vb-to-csharp)