---
post_number: "0300"
title: "Nested Family Instance"
slug: "nested_family_instance"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'levels', 'parameters', 'revit-api']
source_file: "0300_nested_family_instance.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0300_nested_family_instance.html"
---

### Nested Family Instance

We already discussed a few questions related to nested families, on
[exploring nested instance geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html)
and
[creating a nested family](http://thebuildingcoder.typepad.com/blog/2009/11/nested-family.html) using the family API.
Here is another simple question that may arise in the context of family nesting.
It also gives me one of the rare opportunities to present at least a few lines of VB code.

**Question:** Is there a way of testing whether a family instance in a project file is nested inside another, i.e. does it have a parent family instance?

I have a nested family that consists of two families, a computer and a power outlet.

The following code is returning both the instances:

```
  Dim FamilyInstanceSet As ElementSet _
    = oRevApp.App.Create.NewElementSet

  Dim familyInstanceItor As ElementIterator _
    = m_RevitDoc.Elements(GetType(Elements.FamilyInstance))

  familyInstanceItor.Reset()

  While familyInstanceItor.MoveNext()

    Dim Instance As Elements.FamilyInstance _
      = DirectCast(familyInstanceItor.Current, _
        Elements.FamilyInstance)

    FamilyInstanceSet.Insert(Instance)

  End While
```

In my model I have several power outlets.
I need to be able to test the outlets returned, and not count those that are nested within another family.
The family parameter of the nested outlet is set to shared, so that they are exposed at project level, and so can be tagged.

**Answer:** I verified that the retrieval of the document elements returns both the top level and the nested family instances, just like you say.
The original version of your test command reports the following three instances:

```
1 Specialty Equipment-Cupboard-1200x700
2 A1 metric
3 Electrical Fixture-Gen-Double Socket-Double
```

Exploring the model with RvtMgdDbg and checking the Revit API documentation, I noticed the two family instance properties SubComponents and SuperComponent which return information about its nesting relationships:

- SubComponents: retrieve the sub components of the family instance. If it has no sub components, it returns null.- SuperComponent: retrieve the super component of the family instance. If it has no super component, it returns null.

I expanded your code very slightly to check the SuperComponent property for a null value, and assume that that means that the family instance is a top level one, like this:
```vbnet
Dim oRevApp As Application \_
  = commandData.Application

Dim oRevitDoc As Document \_
  = oRevApp.ActiveDocument

Dim FamilyInstanceSet As ElementSet \_
  = oRevApp.Create.NewElementSet

Dim familyInstanceItor As ElementIterator \_
  = oRevitDoc.Elements(GetType(Elements.FamilyInstance))

Dim fndFamilies As String = ""

Dim k As Integer

familyInstanceItor.Reset()

While familyInstanceItor.MoveNext()

  Dim instance As Elements.FamilyInstance \_
    = DirectCast(familyInstanceItor.Current,  \_
      Elements.FamilyInstance)

  Dim s As String = "nested"

  If instance.SuperComponent Is Nothing Then
    s = "top level"
  End If

  k += 1

  fndFamilies &= CStr(k) & " " & instance.Name \_
    & ": " & s & vbCrLf

End While

MsgBox("Found families" & vbCrLf & vbCrLf & fndFamilies)
```

With this enhancement, the message box now displays:

```
1 Specialty Equipment-Cupboard-1200x700: top level
2 A1 metric: top level
3 Electrical Fixture-Gen-Double Socket-Double: nested
```

So the nested electrical fixtures are correctly identifies as such.