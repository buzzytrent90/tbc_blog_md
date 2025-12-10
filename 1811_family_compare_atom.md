---
post_number: "1811"
title: "Family Compare Atom"
slug: "family_compare_atom"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'parameters', 'references', 'revit-api', 'sheets', 'transactions', 'vbnet']
source_file: "1811_family_compare_atom.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1811_family_compare_atom.html"
---

### Swissbau Basel and Comparing Family Part Atoms
Here is a last-minute announcement that I will be attending the Swissbau Basel on Wednesday, a quick hint to answer a support case that just came in today, a forum thread issue, and a pointer to a drum solo:
- [Attending Swissbau Basel on Wednesday](#1)
- [Comparing families using part atoms](#2)
- [Maximum area rectangle in polygon](#3)
- [Neil Peart drum solo](#4)
#### Attending Swissbau Basel on Wednesday
I am attending [Swissbau Basel](https://www.swissbau.ch) on Wednesday, January 15.
Please reach out if you would like to meet there.
Thank you, and looking forward to seeing you!
#### Comparing Families using Part Atoms
I recently discussed [comparing symbols and comparison operators](https://thebuildingcoder.typepad.com/blog/2019/12/comparing-symbols-and-comparison-operators.html),
including pointers to previous ponderings
on [defining your own key for comparison purposes](https://thebuildingcoder.typepad.com/blog/2012/03/great-ocean-road-and-creating-your-own-key.html#2)
and [tracking element modification](https://thebuildingcoder.typepad.com/blog/2016/01/tracking-element-modification.html#5.1).
In a [comment on that post](https://thebuildingcoder.typepad.com/blog/2019/12/comparing-symbols-and-comparison-operators.html#comment-4718582177),
Matt Taylor suggested an effective solution to compare family definitions using the Revit API `ExtractPartAtomFromFamilyFile` method:
I would say that this is a good candidate for
the [ExtractPartAtomFromFamilyFile](https://www.revitapidocs.com/2020/1f2c631b-2733-0aa7-051c-42bccb07f05e.htm)
and [ExtractPartAtom methods](https://www.revitapidocs.com/2020/d477cf8f-0dfe-4055-a787-315c84ef5530.htm).
You can call those on your family and compare the results they return.
The article on [Extract Part Atom Revisited](https://thebuildingcoder.typepad.com/blog/2010/09/extract-part-atom-revisited.html) shows
how they can be invoked.
As an example, consider a basic column family with width and depth parameters, both set to 600mm, and a type named '600x600'.
Load that into a project and change the width to 590.
Export the part atoms of each, e.g., like this in VB.NET:
```vbnet
Imports Autodesk.Revit
Imports Autodesk.Revit.Attributes

Public Class InternalExportPartAtoms
Implements UI.IExternalCommand
Public Function Execute(
ByVal commandData As UI.ExternalCommandData,
ByRef message As String,
ByVal elements As DB.ElementSet) _
As UI.Result Implements UI.IExternalCommand.Execute
Dim app As ApplicationServices.Application = commandData.Application.Application
Dim doc As DB.Document = commandData.Application.ActiveUIDocument.Document
Dim family As DB.Family = TryCast(doc.GetElement(New DB.ElementId(4568558)), DB.Family)
Dim familyName As String = family.Name
Dim logFileFolder As String = "C:\Users\Desktop\PartAtoms\"
app.ExtractPartAtomFromFamilyFile(logFileFolder & familyName & ".rfa",
logFileFolder & familyName & "-file.xml")
family.ExtractPartAtom(logFileFolder & familyName & "-family.xml")
Return UI.Result.Succeeded
End Function
End Class
```
Then do
a [diff](https://en.wikipedia.org/wiki/Diff) on the two outputs,
e.g., [using `diffchecker.com`](https://www.diffchecker.com/Unw6nrB2):
![Diff between family part atoms](img/diff_between_family_part_atoms.jpg "Diff between family part atoms")
This won't catch all tampering, but it's a decent tool for comparison.
Many thanks to Matt for this invaluable tip!
#### Maximum Area Rectangle in Polygon
Completely unrelated to the Revit API, an interesting question popped up today in
the [discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [drawing maximum area rectangle inside a polygon](https://forums.autodesk.com/t5/revit-api-forum/wanted-to-draw-maximum-area-rectangle-inside-a-polygon/m-p/9248884)
– it is such a nice question, I picked it up anyway:
\*\*Question:\*\* We are working on a project related to the construction industry, building an iPad application, where we need to find a MAXIMUM POSSIBLE RECTANGLE INSIDE THE GIVEN POLYGON lines. We totally approached around 100's of ways, but till now we are not getting a right solution. Any suggestions will be really helpful!
![Maximum area rectangle in polygon](img/maximum_area_rectangle_in_polygon.png "Maximum area rectangle in polygon")
\*\*Answer:\*\* I searched the Internet for [maximum inscribed area rectangle in polygon](https://duckduckgo.com/?q=maximum+inscribed+area+rectangle+in+polygon).
The first hit I found was this same question of yours in a different forum with several very helpful suggestions for solving it,
on [How to find the maximum-area-rectangle inside a convex polygon](https://gis.stackexchange.com/questions/59215/how-to-find-the-maximum-area-rectangle-inside-a-convex-polygon).
Another hit shares a proven working solution for
the [largest rectangle in a polygon – an approximation algorithm for finding the largest rectangle inside a non-convex polygon](https://d3plus.org/blog/behind-the-scenes/2014/07/08/largest-rect).
The latter solution also includes several references to scientific papers on the topic.
#### Neil Peart Drum Solo
In closing, you may enjoy
this [pretty cool drum solo by Neil Peart live in Frankfurt](https://youtu.be/LWRMOJQDiLU):

[Neil Peart](https://en.wikipedia.org/wiki/Neil_Peart) was
a fellow Canadian – you may be surprised to hear that I am one too, besides other things – and passed away last week, on January 7, 2020.