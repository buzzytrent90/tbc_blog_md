---
post_number: "0211"
title: "New Sweep and References"
slug: "new_sweep_references"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'references', 'revit-api', 'vbnet', 'walls']
source_file: "0211_new_sweep_references.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0211_new_sweep_references.html"
---

### New Sweep and References

Here is a
[question](http://thebuildingcoder.typepad.com/blog/2009/02/creating-a-wall-with-a-sloped-profile.html?cid=6a00e553e1689788330120a55da0b8970c#comment-6a00e553e1689788330120a55da0b8970c)
posted by Mike Cotreau of
[Langan Engineering & Environmental Services](http://www.langan.com)
on how to create a new sweep using a reference array, which provided a welcome opportunity to explain a little about the use of references in the Revit API:

**Question:**
I want to use the NewSweep method to create a 3D curb line that follows along a path of 3D points.
The NewSweep method can be invoked in two ways, one uses a ReferenceArray and the second uses a CurveArray.
There is an example provided in the VSTA Family Sample code that illustrates the CurveArray version and I have used that code to layout an entire curbline without issues.
Unfortunately, I need to use the first version that allows for a sweep to follow a 3D path using the ReferenceArray version.
I keep getting an error saying the object is not referenced, and I am clueless as to why.
I am using VB.NET and here is the code I modified from the sample code:

```csharp
Dim arrarr As CurveArrArray = m\_revit.Create.NewCurveArrArray()
Dim arr As CurveArray = m\_revit.Create.NewCurveArray()
'\*\*\*\*\*\* This creates the section for the curb \*\*\*\*\*\*\*\*
Dim normal As XYZ = XYZ.BasisZ
Dim sketchPlane As SketchPlane = CreateSketchPlane(normal, XYZ.Zero)
Dim pnt1 As XYZ = m\_revit.Create.NewXYZ(0, 0, 0)
Dim pnt2 As XYZ = m\_revit.Create.NewXYZ(0, 0.5, 0)
Dim pnt3 As XYZ = m\_revit.Create.NewXYZ(0.5, 0.5, 0)
Dim pnt4 As XYZ = m\_revit.Create.NewXYZ(0.5, -1, 0)
Dim pnt5 As XYZ = m\_revit.Create.NewXYZ(0, -1, 0)
arr.Append(m\_revit.Create.NewLineBound(pnt1, pnt2))
arr.Append(m\_revit.Create.NewLineBound(pnt2, pnt3))
arr.Append(m\_revit.Create.NewLineBound(pnt3, pnt4))
arr.Append(m\_revit.Create.NewLineBound(pnt4, pnt5))
arr.Append(m\_revit.Create.NewLineBound(pnt5, pnt1))
arrarr.Append(arr)
Dim profile As SweepProfile = m\_revit.Create.NewCurveLoopsProfile(arrarr)
Dim curves As CurveArray = m\_revit.Create.NewCurveArray()
Dim refArray As ReferenceArray = m\_revit.Create.NewReferenceArray
Dim pt1 As XYZ = m\_revit.Create.NewXYZ(-265.9978, -358.3954, 0)
Dim pt2 As XYZ = m\_revit.Create.NewXYZ(-265.9374, -340.4006, 0)
Dim curve As Curve = m\_revit.Create.NewLineBound(pt1, pt2)
curves.Append(curve)
refArray.Append(curve.Reference)
'Create the Sweep
Dim sweep1 As Sweep = m\_creationFamily.NewSweep(True, refArray, profile, 0, ProfilePlaneLocation.Start)
' move to proper place
Dim transPoint1 As XYZ = m\_revit.Create.NewXYZ(11, 0, 0)
m\_familyDocument.Move(sweep1, transPoint1)
```

An exception is thrown on the NewSweep method. I have also tried building the Reference array using code like this without any luck:

```
refArray.Append(curve.get_EndPointReference(0))
refArray.Append(curve.get_EndPointReference(1))
```

Any help you can provide that allows me to sweep along a 3D path would be GREATLY appreciated.

**Answer:**
NewSweep is used in the family document to create a new sweep.
The overloads you refer to for creating a new sweep form are:

- NewSweep( Boolean, ReferenceArray, SweepProfile, Int32, ProfilePlaneLocation ), using an array of selected references as a 3D path.- NewSweep( Boolean, CurveArray, SketchPlane, SweepProfile, Int32, ProfilePlaneLocation ), using a path of curve elements.

The three last arguments are the same for both overloads, so these are presumably not cause of the problem:

- SweepProfile profile,- int profileLocationCurveIndex, and- ProfilePlaneLocation profilePlaneLocation.

The API documentation specifies the reference array passed in to define the path in the first overload as follows:

- ReferenceArray path:
  The path of the sweep.
  The path should be reference of curve or edge obtained from existing geometry.

The cause of your problem is almost certainly the fact that the curve that you are trying to obtain references from is not part of any existing geometry, since you created it on the fly.

To understand the difference and the importance of this, here are some private hypothetical musings of mine on what references actually mean within Revit.
I have never seen any hard documentation of this, but I am gradually getting the following idea:

#### References

All of objects living in the Revit API Geometry namespace are transient in-memory objects.
The non-transient Revit BIM model resides in the database.
To query the geometry of BIM objects, transient geometry is generated on the fly from their parametric representations.
If a new BIM object B is to be generated and stored in the database, such as your sweep, and it needs to reference existing BIM geometry in any way, for instance some edge of an existing object A, then a reference can be generated to provide a hook from the new object B definition through the transient geometry back to the underlying BIM object A.
Later on, when the BIM is saved, the transient geometry has all disappeared, but the reference ensures that B knows that it depends on the geometry and location of A in some way.
If A is moved or modified, B can adapt.

In your sample, you are creating a new line between two fixed points pt1 and pt2.
This is all transient geometry, and not referring back to any BIM object, so there is no way that the resulting curve can contain or provide a valid reference to anything.

The question is why you are trying to use the overload requiring a reference array at all?
If your input data is fixed, you should use the curve array option instead.

By the way, for completeness sake, prompted by your query, I had a look at the VSTA family samples.
They are provided in the Revit SDK VSTA Samples sub-folder, in the family file Revit\_VSTA\_Family\_Samples.rfa.
I see the code you based yours on, in the subroutine CreateSweep() of the GenericModelCreation project.
This project is also provided as one of the standard SDK samples as well, albeit only in a C# version.

I hope this answers your question. Good luck!