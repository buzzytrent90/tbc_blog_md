---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.441598'
original_url: https://thebuildingcoder.typepad.com/blog/1639_linked_inters_filt.html
post_number: '1639'
reading_time_minutes: 7
series: general
slug: linked_inters_filt
source_file: 1639_linked_inters_filt.md
tags:
- elements
- filtering
- geometry
- parameters
- revit-api
- sheets
- transactions
- views
title: Linked Inters Filt
word_count: 1303
---

### Using Intersection Filter with Linked File
We recently
discussed [filtering for intersecting elements](http://thebuildingcoder.typepad.com/blog/2018/03/create-2d-arc-and-filter-for-intersecting-elements.html).
Here is a closely related issue with an additional twist that I discussed
with Gustav Blom, structural engineer at [Rambøll](http://www.ramboll.no) in Norway:
- [Determining elements intersecting mass in a linked file](#2)
- [Coding suggestions and transformations](#3)
- [Solution by applying transformations](#4)
![Rambøll](img/ramboell1.png)
#### Determining Elements Intersecting Mass in a Linked File
\*\*Question:\*\* I am implementing an add-in that determines all model elements inside a mass in a linked file \*MassModel.rvt\*.
I have used a filtered element collector with an `ElementIntersectsSolidFilter` using the solid representation of the mass as argument.
Here is my main filtering code:
```vbnet
Public Sub Run()
Dim oWrite1 As IO.StreamWriter
oWrite1 = IO.File.CreateText("C:/ . . . /ObjectLocation.txt")
' Collect linked files
Dim linkedfilefilter As ElementCategoryFilter _
= New ElementCategoryFilter(BuiltInCategory.OST_RvtLinks)
Dim linkedfilecollector As FilteredElementCollector _
= New FilteredElementCollector(m_doc) _
.WherePasses(linkedfilefilter) _
.WhereElementIsNotElementType
Dim options As Options = New Options()
Dim tx As Transaction = New Transaction(m_doc)
tx.Start("Object Locate")
If linkedfilecollector.Count > 0 Then
For Each linkedfile As RevitLinkInstance In linkedfilecollector
If linkedfile.GetLinkDocument.Title.Equals("MassModel.rvt") Then
Try
' Find all elements with OBJ-LOCATION-RDK
' Parameter And erase previous values
Dim collector As FilteredElementCollector _
= New FilteredElementCollector(m_doc) _
.WhereElementIsNotElementType _
.WhereElementIsViewIndependent
For Each el As Element In collector
If el.Category IsNot Nothing And el.Parameters.Size > 0 Then
Dim ElementParameters As ParameterSet = el.Parameters
For Each elparam As Parameter In ElementParameters
If elparam.Definition.Name = "OBJ-LOCATION-RDK" Then
elparam.Set("")
oWrite1.WriteLine(el.Category.Name)
End If
Next
End If
Next
Dim linkedmassCatFilter As ElementCategoryFilter _
= New ElementCategoryFilter(BuiltInCategory.OST_Mass)
Dim massCollector As FilteredElementCollector _
= New FilteredElementCollector(linkedfile.GetLinkDocument) _
.WhereElementIsNotElementType() _
.WherePasses(linkedmassCatFilter)
Dim populatedcounter As Integer = 0
For Each mass As Element In massCollector
Dim solSet As Generic.IEnumerable(Of Solid) _
= mass.Geometry(options).OfType(Of Solid)()
For Each potSol As Solid In solSet
If (potSol IsNot Nothing And Not potSol.Edges.IsEmpty) Then
Dim elementintersectsolidfilter As ElementIntersectsSolidFilter _
= New ElementIntersectsSolidFilter(potSol)
Dim intersectingelems As FilteredElementCollector _
= New FilteredElementCollector(m_doc) _
.WherePasses(elementintersectsolidfilter)
oWrite1.WriteLine("intersecting elems: " _
+ intersectingelems.Count.ToString)
For Each intersectingelem As Element In intersectingelems
Dim ElementParameters As ParameterSet _
= intersectingelem.Parameters
For Each param As Parameter In ElementParameters
If param.Definition.Name = "OBJ-LOCATION-RDK" Then
If Not param.AsString = "" Then
param.Set(param.AsString + "+" + mass.Name)
Else
param.Set(mass.Name)
populatedcounter += 1
End If
End If
Next
Next
End If
Next
Next
MsgBox("Populated parameter OBJ-LOCATION-RDK for " _
+ populatedcounter.ToString + " model elements")
Exit For
Catch ex As Exception
End Try
Else
MsgBox("No linked file by name of MassModel.rvt found")
End If
Next
Else
MsgBox("No linked files in project. " _
+ "Please link a file named MassModel.rvt into project")
End If
tx.Commit()
oWrite1.Close()
End Sub
```
The strange thing is this: it only returns the elements that have been offset or disjoined from their work plane.
Is there a simpler way to achieve what I am trying to do, like the `geometry.DoesIntersect` node in Dynamo?
#### Coding Suggestions and Transformations
\*\*Answer:\*\* I'll begin with a comment or two on your sample code:
First, it is cleaner and safer and easier to encapsulate transactions in a `using` clause:
[using `using` automagically disposes and rolls back](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html).
Refer to The Building Coder topic group for more
on [handling transactions and transaction groups](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.53).
Secondly, you can access a parameter on an element directly by name.
It is better to use a display name independent method when possible, but it works well if you can guarantee that the given parameter name is unique on that element.
Then you can thus replace the following lines of code:
```vbnet
Dim ElementParameters As ParameterSet = el.Parameters
For Each elparam As Parameter In ElementParameters
If elparam.Definition.Name = "OBJ-LOCATION-RDK" Then
elparam.Set("")
oWrite1.WriteLine(el.Category.Name)
End If
Next
```
They can be replaced by something like:
```csharp
IList plist = e.GetParameters("OBJ-LOCATION-RDK")
Parameter elparam = plist[0]
elparam.Set("")
```
That would be faster, more efficient, easier to read and understand.
It has nothing to do with your question, though, does it?
Looking closer at your specific question:
I was under the impression that all Dynamo code is public domain and open source, so you can explore the implementation of the `DoesIntersect` node yourself.
Have you checked out that possibility?
The mass element lives in `linkedfile.GetLinkDocument`, and so do the solids `potSol` that you extract from it. Is that correct?
The intersecting elements that you are searching for live in the project document `m_doc`, correct?
What is the transformation from the linked document into the project document?
Have you taken account of that?
It is interesting to hear that the intersection works in some cases, but not in others: 'elements offset or disjoined from their work plane'.
Maybe that is affecting their transformation in some way.
I recently discussed [filtering for intersecting elements](http://thebuildingcoder.typepad.com/blog/2018/03/create-2d-arc-and-filter-for-intersecting-elements.html) in
general.
Another, even more relevant discussion
concerns [linked file element intersection solid geometry](https://forums.autodesk.com/t5/revit-api-forum/linked-file-element-intersection-solid-geometry/m-p/7861611).
There, I make the following suggestion:
You have two solids, `A` in your main project `P` and `B` in your linked project `Q`.
- Determine the transformation `T` from the linked document `Q` coordinates to `P`'s.
- Open the linked project `Q` and retrieve the solid `Sb` of `B`.
- Transform it to `P`'s coordinate space: `T \* Sb`.
- Retrieve the solid `Sa` of `A`.
- Calculate the intersection of `T\*Sb` with `Sa`.
Now you can use an element intersection filter based on the intersection result like in the discussion mentioned before.
The transformation of `Sb` can be obtained using
the [SolidUtils.CreateTransformed method](http://www.revitapidocs.com/2018.1/22592761-f39c-4f53-d33b-6c21a4fa9d2d.htm).
Please check that out as well.
#### Solution by Applying Transformations
\*\*Response:\*\* You are correct; I had not considered the transformation of the linked mass elements, and it was only by dumb luck that every time I offset or disjoined a model element it would intersect with the default position of the linked masses.
Once the transform was added to the code it worked just as expected.
You are welcome to share this on your blog.
Here is the relevant code:
```vbnet
' Determine transform of linked file in main project
Dim transform As Transform = linkedfile.GetTotalTransform
For Each mass As Element In massCollector
Dim massSolids As Generic.IEnumerable(Of Solid) _
= mass.Geometry(options).OfType(Of Solid)()
For Each massSolid As Solid In massSolids
If (massSolid IsNot Nothing _
And Not massSolid.Edges.IsEmpty) Then
' Apply transform to mass
Dim transformedSolid As Solid = SolidUtils _
.CreateTransformed(massSolid, transform)
Dim elementIntersectsSolidFilter As _
ElementIntersectsSolidFilter = New _
ElementIntersectsSolidFilter(transformedSolid)
Dim intersectingElems As FilteredElementCollector _
= New FilteredElementCollector(m_doc) _
.WhereElementIsNotElementType _
.WhereElementIsViewIndependent _
.WherePasses(elementIntersectsSolidFilter)
For Each intersectingElem As Element _
In intersectingElems
Dim elParams As IList(Of Parameter) _
= intersectingElem.GetParameters(
"Mass Intersections")
If elParams.Count > 0 Then
Dim elParam As Parameter = elParams(0)
If elParam.AsString = "" Then
elParam.Set(mass.Name)
populatedCounter += 1
Else
elParam.Set(elParam.AsString _
+ "+" + mass.Name)
End If
End If
Next
End If
Next
Next
```
Apart from cleaning it up a bit, the only changes made from the original case are in the lines beneath the comments.
![Rambøll](img/ramboell2.png)
Many thanks to Gustav for raising this interesting issue and sharing his solution!