---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: qa
optimization_date: '2025-12-11T11:44:13.639985'
original_url: https://thebuildingcoder.typepad.com/blog/0262_dimension_types.html
post_number: '0262'
reading_time_minutes: 6
series: elements
slug: dimension_types
source_file: 0262_dimension_types.htm
tags:
- csharp
- elements
- parameters
- references
- revit-api
- views
title: Distinguish Different Dimension Types
word_count: 1110
---

### Distinguish Different Dimension Types

I am back in Europe again after spending most of the weekend travelling, similarly to
[Kean](http://through-the-interface.typepad.com/through_the_interface/2009/12/asynchronous-messages-in-f-and-autocad.html) but
in the opposite direction.
In his last post, Kean summarised some interesting views on the
[evolution of Autodesk University](http://through-the-interface.typepad.com/through_the_interface/2009/12/the-evolution-of-autodesk-university.html).

Returning to the Revit API, here is a nice can-do solution presenting a workaround created by my colleague Joe Ye of Autodesk and Henrik Bengtsson of
[Lindab](http://www.lindab.se) on
how to distinguish between the different kinds of dimension types.

**Question:** I am creating a drafting view with dimensions and I have a small problem.
The function listed below works all right, but I had to hard code the dimension type name to look for.
The if statement comparing the type name with the hardcoded string "Arrow - 2.5mm Arial" should not be needed.

This is what the code should do:

1. Loop through the existing dimension types.
   If the one I am looking for is found, return it directly.
   The first time this function is called in a new document, it should not find anything.- The second loop should simply pick the first one, and then exit the loop.
     Unfortunately when the first item in the loop is used, I cannot set Dimension.DimensionType.
     That is done outside this routine.- Duplicate the type picked in step 2, update its parameters appropriately, and return it.

In some way or another, there must be different dimension types included in the Document.DimensionTypes collection.
When I grab the first one, it is unfortunately of the wrong type.
How can I find a correct type to duplicate?

I think that I use Linear Dimension Type, but there seem to be others as well.

I have looped though the parameters but I cannot find any that would tell me the dimension type.
I see one named Linear\_Dim\_type, but it returns 1 in all cases, so that is no help.
```vbnet
Public Function GetDimensionType( \_
  ByVal doc As Document, \_
  ByVal name As String, \_
  ByVal size As Double) \_
As Symbols.DimensionType

  Dim dt As Symbols.DimensionType = Nothing

  Try

    For Each itm As Symbols.DimensionType In doc.DimensionTypes

      If itm.Name = name Then

        dt = itm
        Return dt

      End If

    Next

    If dt Is Nothing Then

      Dim tmpdt As Symbols.DimensionType = Nothing

      For Each itm As Symbols.DimensionType In doc.DimensionTypes

        If itm.Name = "Arrow - 2.5mm Arial" Then ' hardcoded string
          tmpdt = itm
          Exit For
        End If

      Next

      dt = tmpdt.Duplicate(name)

      dt.Parameter( \_
        Parameters.BuiltInParameter.TEXT\_SIZE) \_
        .Set(mmTofoot(size))
      dt.Parameter( \_
        Parameters.BuiltInParameter.TEXT\_DIST\_TO\_LINE) \_
        .Set(mmTofoot(1))

    End If

  Catch ex As Exception
  End Try

  Return dt

End Function
```

Here is a snippet of code showing how I call this routine:
```csharp
Dim ref1 As New Reference
Dim ref2 As New Reference
Dim refArray As New ReferenceArray
refArray.Append(ref1)
refArray.Append(ref2)
Dim dimensionLine As Line \_
  = app.Create.NewLine(refP1, refP2, True)

Dim dimension As Dimension \_
  = doc.Create.NewDimension(view, dimensionLine, refArray)

' this fails if the wrong type is returned:
dimension.DimensionType \_
  = GetDimensionType(doc, "Lindab-2.0", 2.0)
```

**Answer:** There are three kinds of dimension types

- Angular- Linear- Radial

I agree that you probably trying to assign a wrong type of dimension style to the dimension.

So the question is how to distinguish them correctly.

I just investigated the properties and parameters of dimension style elements.
I do not see any single value that would enable me to distinguish any one of these from the other two.
Since Revit can distinguish them internally, I assume it has some additional data that we do not see in the API.

However, the three kinds of styles have different collections of parameters, and these can be used to distinguish them.

For example, the 'Dimension String Type' parameter only occurs on a Linear style.
Therefore, if we can read a valid parameter value for this parameter, we know that it is a linear dimension style:

![Linear dimension style element properties](img/dim_style_linear.png)

To distinguish between the other two styles, I found that the 'Centerline Style', 'Centerline Pattern', 'Centerline Tick Mark' and 'Interior Tick Mark' parameters only occur on an angular style:

![Angular dimension style element properties](img/dim_style_angular.png)

A radial dimension style does not have these:

![Radial dimension style element properties](img/dim_style_radial.png)

**Response:** I checked the return values of these parameters and unfortunately they all seem to exist.
what I mean is that all dimension types have these...
I was hoping that some of them would return 'Nothing'.

**Answer:** If you access these parameters directly by calling Element.get\_Parameter with a string name or built-in parameter argument, this might not work as expected.
A specific parameter, for example Linear\_Dim\_Type, may well be invisible for Angular and Radial types, but it may still exist.

Please retrieve the parameters by iterating the ParameterSet returned by DimensionType.Parameters.
This collection will only return the parameters visible in the Properties Dialog.
In this way we can discover if the specific target parameter is valid in this ParameterSet.
Use the same method to detect if specific parameters exist in the remaining two dimension types.

**Response:** Here is a final version of the code.
It works for me in my project.
That does not guarantee that it will work in another environment.
So please just use this as a basic idea and test it in your context:
```vbnet
Public Function GetDimensionType( \_
  ByVal doc As Document, \_
  ByVal name As String, \_
  ByVal size As Double) \_
As Symbols.DimensionType

  Dim dt As Symbols.DimensionType = Nothing
  Try

    For Each itm As Symbols.DimensionType In doc.DimensionTypes

      If itm.Name = name Then

        dt = itm
        Return dt

      End If

    Next
    If dt Is Nothing Then

      Dim tmpdt As Symbols.DimensionType = Nothing
      For Each itm As Symbols.DimensionType In doc.DimensionTypes

        If Not tmpdt Is Nothing Then

          Exit For

        End If
        For Each p As Parameter In itm.Parameters

          Dim id As Parameters.InternalDefinition = p.Definition
          If id.BuiltInParameter = Parameters.BuiltInParameter.LINEAR\_DIM\_TYPE Then

            tmpdt = itm
            Exit For

          End If

        Next

      Next
      dt = tmpdt.Duplicate(name)

      dt.Parameter( \_
        Parameters.BuiltInParameter.TEXT\_SIZE) \_
        .Set(mmTofoot(size))

      dt.Parameter( \_
        Parameters.BuiltInParameter.TEXT\_DIST\_TO\_LINE) \_
        .Set(mmTofoot(1))

    End If

  Catch ex As Exception
  End Try

  Return dt

End Function
```

If the lines above are truncated in your browser, simply copy and paste them to an editor.