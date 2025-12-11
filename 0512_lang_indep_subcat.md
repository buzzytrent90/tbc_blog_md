---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.079348'
original_url: https://thebuildingcoder.typepad.com/blog/0512_lang_indep_subcat.html
post_number: '0512'
reading_time_minutes: 10
series: general
slug: lang_indep_subcat
source_file: 0512_lang_indep_subcat.htm
tags:
- csharp
- elements
- family
- levels
- parameters
- revit-api
- selection
- transactions
- walls
title: Language Independent Subcategory Creation
word_count: 1983
---

### Language Independent Subcategory Creation

Here is a snapshot of a lengthy conversation that I have been leading with Gregory Mertens of
[mertens3d.com](http://www.mertens3d.com) for
a while, starting out with a language independence issue and touching on other topics such as category and subcategory creation, use of built-in parameters, use of debug assertions and exception handling.

We looked at a couple of category related issues in the past:

- [Category comparison](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html)- [Family categories](http://thebuildingcoder.typepad.com/blog/2009/07/get-and-set-family-category-and-parameters.html)- [Categories and parameter bindings](http://thebuildingcoder.typepad.com/blog/2009/09/adding-a-category-to-a-parameter-binding.html)- [System versus user defined categories](http://thebuildingcoder.typepad.com/blog/2009/10/system-versus-user-family-category.html)- [Complete list of categories](http://thebuildingcoder.typepad.com/blog/2010/05/categories.html)

We have not yet looked at any examples of creating subcategories.
Top level categories cannot be created, by the way.

Here is an example of creating subcategories, eliminating a language dependence in the process, and some thoughts on this, raised by Gregory:

**Question:** I am using the following code to create new line subcategories
(to see the truncated lines of VB code in full, please copy to a text editor):
```vbnet
Public Sub createNewLineSubCategory( \_
  ByRef newSubCatName As String, \_
  ByRef thisSubColor As Autodesk.Revit.DB.Color)

  Dim oneCat As Autodesk.Revit.DB.Category

  Dim mySubCat As Autodesk.Revit.DB.Category

  Dim docTransaction As New Autodesk.Revit.DB.Transaction(ptr2Doc)

  For Each oneCat In ptr2Doc.Settings.Categories
    If oneCat.Name = "Lines" Then

      'we need to cruise through each subcategory and make sure it doesn't
      'already exist

      ' Dim allSubCategories As Autodesk.Revit.DB.CategoryNameMap = oneCat.SubCategories

      ' Dim oneSubCategory As Autodesk.Revit.DB.Category
      'we know it doens't exist, because we tested for it prior to this routine

      Try
        docTransaction.SetName("allSubCategories")
        docTransaction.Start()
        mySubCat = ptr2Doc.Settings.Categories.NewSubcategory(oneCat, newSubCatName)
        mySubCat.LineColor = thisSubColor
        docTransaction.Commit()

      Catch ex As Exception
        MsgBox(ex.ToString)
      End Try

      Exit For
    End If
  Next

End Sub
```

It works great when run on an English installation of Revit.

However, it doesn't work when I run it on an Italian install.
It looks like what is "Lines" to English users is "Linee" (2 e's) to Italian.

I suspect that I'm not coding this correctly.
Surely I am not supposed to have a separate check for each language.

Could you comment on the correct way to code so that all language versions of Revit work correctly (at least relative to this issue)?

**Answer:** To put it briefly,

1. Do not use the language dependent string comparison oneCat.Name = "Lines".- Find out what the built-in category is instead ... OST\_Lines?- Do not iterate over the entire collection of categories, use get\_Item instead: Categories.Item Property (BuiltInCategory).- Send me your final code once you have solved this, please.

**Response:** Thanks for the very quick and helpful response.

Here is some new code that works:
```vbnet
Public Sub createNewLineSubCategory( \_
  ByRef newSubCatName As String, \_
  ByRef newSubCatColor As Autodesk.Revit.DB.Color)

  Dim lineCat As Autodesk.Revit.DB.Category

  Dim lineSubCat As Autodesk.Revit.DB.Category

  Dim docTransaction As New Autodesk.Revit.DB.Transaction(ptr2Doc)

  lineCat = ptr2Doc.Settings.Categories.Item(BuiltInCategory.OST\_Lines)

  Try
    docTransaction.SetName("hatch22 - Create SubCategory")
    docTransaction.Start()
    lineSubCat = ptr2Doc.Settings.Categories.NewSubcategory(lineCat, newSubCatName)
    lineSubCat.LineColor = newSubCatColor
    docTransaction.Commit()

  Catch ex As Exception
    MsgBox(ex.ToString)
  End Try

End Sub
```

That (of course) led to other issues:
I had to look through the rest of my code to see if there were any other places where I was using language dependant comparisons.
One of them I solved like this:
```vbnet
Public Sub changeLineStyleGeneric( \_
  ByVal thisElement As Autodesk.Revit.DB.Element, \_
  ByRef newLineStyleID As Autodesk.Revit.DB.ElementId)

  Dim param As Parameter
  Dim parameters As ParameterSet

  If Not IsNothing(newLineStyleID) Then
    parameters = thisElement.Parameters
    For Each param In parameters

      If param.Definition.ParameterGroup = BuiltInParameterGroup.PG\_GRAPHICS \_
         And \_
        Not param.AsElementId.IntegerValue = -1 Then

        Try
          param.Set(newLineStyleID)
        Catch ex As Exception
          MsgBox(ex.ToString)
        End Try

      End If
    Next
  End If

End Sub
```

Here is how I determined the right parameter, without searching for the language dependent string "linestyle":
```csharp
  If param.Definition.ParameterGroup = BuiltInParameterGroup.PG\_GRAPHICS \_
    And Not param.AsElementId.IntegerValue = -1 Then
```

The portion above doesn't feel very clean to me.
Thoughts?

I don't feel confident that my routine won't come across some other (undesired) parameter that has no ID and is in PG\_GRAPHICS.
Am I somehow guaranteed that my test will come up with one and only one parameter (for that test)?
(I suppose one thing I should do is research what it means to be 'PG\_GRAPHICS').

I came up with the test while debugging.
At first I was only testing for PG\_GRAPHICS.
It turned out that sometimes this caused errors (not all the time).
I traced down the error and discovered that there were some parameters that met the PG\_GRAPHICS test, but did not have an element ID (they were = -1).

What bothers me is that I don't know that other (possibly future) parameters won't pass both tests.
Does that make sense?
It seems to me there should be an absolute way to know if I've come across a parameter that holds the line style.

Oh ... it just occurred to me.
Maybe I could check.
Maybe after getting the ID parameter that I suspect points to a line style parameter I could inspect that ID's element and verify that it is in fact a line style.
It seems odd that Revit would work that way internally though.
This does seem a bit inefficient, though.

I just don't know what I don't know here.

It does work, though, so far.

**Answer:** All I can suggest is to add lots of
[assertions](http://en.wikipedia.org/wiki/Assertion_%28computing%29) using
the System.Diagnostics namespace and the Debug.Assert method, and run the code on lots of large and varied test cases.

**Response:** When I started this program (which is based on code I wrote some 15 years ago for AutoCAD), I looked around on the web and was surprised that I couldn't find any other apps (other than hatchkit) that did this.
I would have done it anyway, just for the learning experience, and I'm glad I did.
Wow... there's nothing like reality.

So here's what I'm doing now.
I feel more comfortable with it and it seems to work.
Still testing though:
```vbnet
Public Sub changeLineStyleGeneric( \_
  ByVal thisCurveElement As CurveElement, \_
  ByRef newLineStyleID As ElementId)
  Dim param As Parameter
  Dim parameters As ParameterSet
  If Not IsNothing(newLineStyleID) Then
    parameters = thisCurveElement.Parameters
    For Each param In parameters
      ' a different approach:
      ' to make sure we have the correct parameter,
      ' make sure its value is the same as the known
      ' linestyle ID
      If param.AsElementId = \_
        thisCurveElement.LineStyle.Id Then
        Try
          param.Set(newLineStyleID)
        Catch ex As Exception
          MsgBox(ex.ToString)
        End Try
      End If
    Next
  End If
End Sub
```

I changed my subs input parameter from Autodesk.Revit.DB.Element to Autodesk.Revit.DB.CurveElement.
This allows me to access thisCurveElement.LineStyle.Id which I then compare to the parameter value.
If they are equal, then I am confident that I have the correct parameter and I can assign my new linestyle ID to that parameter.

I looked to see if I could set thisCurveElement.LineStyle.ID directly, but it's read-only.
I didn't see any other method for assigning the ID.

I also tried playing around with "BuiltInParameter.BUILDING\_CURVE\_GSTYLE" but couldn't get anywhere with that.

Makes sense?

**Answer:** Have you checked the curve element parameters using the built-in parameter checker or RevitLookup snoop enum parameters?

As said, you should be able to determine what built-in parameter to use instead of looping through all of the parameters.

I did, in fact, and it looks like "BuiltInParameter.BUILDING\_CURVE\_GSTYLE" may indeed be what you are looking for.

Although, on second thoughts, there are several others as well, which all refer to the line style element id.
In your loop you set all of them, not just one.
Maybe it is necessary to set more than one of them, which is why your loop works and just setting "BuiltInParameter.BUILDING\_CURVE\_GSTYLE" does not.
Here they all are for one detail curve I looked at, all have an ElementId storage type, are read-write, and refer to the GraphicsStyle 'Medium Lines' 185:

- BUILDING\_CURVE\_GSTYLE: Line Style- BUILDING\_CURVE\_GSTYLE\_PLUS\_INVISIBLE: Subcategory- FAMILY\_CURVE\_GSTYLE\_FOR\_2010\_MASS: Subcategory- FAMILY\_CURVE\_GSTYLE\_PLUS\_INVISIBLE: Subcategory- FAMILY\_CURVE\_GSTYLE\_PLUS\_INVISIBLE\_PLUS\_STICK\_SYM: Subcategory- FAMILY\_CURVE\_GSTYLE\_PLUS\_INVISIBLE\_MINUS\_ANALYTICAL: Subcategory- FAMILY\_CURVE\_GSTYLE\_PLUS\_INVISIBLE\_PLUS\_STICK\_SYM\_MINUS\_ANALYTICAL: Subcategory

I would eliminate the try-catch inside the loop, because that should only be used when it is absolutely unavoidable.

You could add a debug assertion to check that the value of the LineStyle.Id property really does equal the newly set one after changing it.

**Response:** Continued thanks.
Regarding the built-in parameter, I was not quite sure how to use it, so I did a search for "BuiltInParameter" in the "Revit 2011 API" chm.
The first item listed had the following sample code:
```vbnet
Public Function FindWithBuiltinParameterID( \_
  ByVal wall As Wall) As Parameter

  ' Use the WALL\_BASE\_OFFSET paramametId
  ' to get the base offset parameter of the wall.
  Dim paraIndex As BuiltInParameter \_
    = BuiltInParameter.WALL\_BASE\_OFFSET

  Dim parameter As Parameter \_
    = wall.Parameter(paraIndex)

  Return parameter
End Function
```

This translates easily to my case below:
```vbnet
Public Sub ChangeLineStyleGeneric( \_
  ByVal thisCurveElement As CurveElement, \_
  ByRef newLineStyleID As ElementId)

  If Not IsNothing(newLineStyleID) Then
    Dim paraIndex As BuiltInParameter \_
      = BuiltInParameter.BUILDING\_CURVE\_GSTYLE

    Dim parameter As Parameter \_
      = thisCurveElement.Parameter(paraIndex)

    parameter.Set(newLineStyleID)
  End If

End Sub
```

This seems to work.

Regarding the try/catch statement:

1. Why should that be avoided?- Is it ok to wrap the whole routine in one?
     It is used on page 25 of the Revit 2011 API Developer Guide.pdf.

Regarding the parameter checker and snoop, I played around a little.
I had not noticed the "Built-in Enums Snoop..." button previously.

Wow... so much to learn!

Does " BuiltInParamsChecker.vb" provide me with any information that "Revit Lookup" does not?

**Answer:**

- Yes, indeed, the developer guide is an invaluable resource and should always be kept close at hand!- Every exception handler is resource intensive and will significantly slow down execution and consume resources.
    An exception handler should be designed to handle unexpected, exceptional cases only.
    [Exceptions should be exceptional](http://www.jacopretorius.net/2009/10/exceptions-should-be-exceptional.html).
    Therefore, it is normally sufficient to implement a single top-level handler at the level of the application mainline, e.g. in the external command Execute method.- The built-in parameter checker provides some addition functionality such as listing the built-in enumeration value, display name, data storage type, database value, string value etc. separately in independent columns and allowing you to sort on any of them.
      That can make it much easier to find something you are looking for, especially in elements with numerous parameters attached.

The discussion and learning continues...

Wait, that's not all!
Here is the complete utility that we are discussing:

#### Hatch22 – a Revit Hatch Utility

Here is a free
[hatch utility](http://mertens3d.com/revit-2011-addin-portal/hatch22) catchily named
[Hatch22](http://mertens3d.com/revit-2011-addin-portal/hatch22) that
Gregory is currently working on.
I love Catch 22 by Joseph Heller, by the way!