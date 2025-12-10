---
post_number: "1282"
title: "Determining the Face Tangent at a Picked Point"
slug: "face_point_tangent"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'references', 'revit-api', 'selection']
source_file: "1282_face_point_tangent.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1282_face_point_tangent.html"
---

### Determining the Face Tangent at a Picked Point

Happy Valentine's Day!

![Happy Valentine's Day!](img/moni_valentine_flowers2.jpg)

I keep repeating how much I love all kinds of geometric problems – besides my sweetheart, of course – merci filmool fier di scheeni Bliemli!

Unfortunately, there are far too few of them around :-)

Happily, Jordi just raised one in the Revit API discussion forum thread on an
[API C# question](http://forums.autodesk.com/t5/revit-api/api-c-question/td-p/5506500),
providing another welcome opportunity to make use of the Face Project and ComputeDerivatives methods:

**Question:**
I'm coding a plugin for Revit 2015 that inserts a family of my choice inside an existing project.

I want to insert the family directly on a face of an existing 3D item.
But depending on the situation I must insert it on different types of faces, such as planar faces or cylindrical.
Problem is, I can't get my code to function on either; I don't know how to check for the type of face prior to my operations.

That's my code for cylindrical faces:

```csharp
  Reference r = uidoc.Selection.PickObject(
    ObjectType.Face, "Please pick a point on a "
    + "face for family instance insertion" );

  Element e = doc.GetElement( r.ElementId );

  GeometryObject obj
    = e.GetGeometryObjectFromReference( r );

  //PlanarFace face = obj as PlanarFace;
  CylindricalFace face = obj as CylindricalFace;

  XYZ p = r.GlobalPoint;
  XYZ v = face.Axis.CrossProduct( XYZ.BasisZ );
  if( v.IsZeroLength() )
  {
    v = face.Axis.CrossProduct( XYZ.BasisX );
  }
  doc.Create.NewFamilyInstance( r, p, v, symbol );
```

And that's my code for planar faces:

```csharp
  Reference r = uidoc.Selection.PickObject(
    ObjectType.Face, "Please pick a point on a "
    + "face for family instance insertion" );

  Element e = doc.GetElement( r.ElementId );

  GeometryObject obj
    = e.GetGeometryObjectFromReference( r );

  PlanarFace face = obj as PlanarFace;
  //CylindricalFace face = obj as CylindricalFace;

  XYZ p = r.GlobalPoint;
  XYZ v = face.Normal.CrossProduct( XYZ.BasisZ );
  if( v.IsZeroLength() )
  {
    v = face.Normal.CrossProduct( XYZ.BasisX );
  }
  doc.Create.NewFamilyInstance( r, p, v, symbol );
```

Almost nothing changes except for the declaration of v with the change: Axis/Normal.

I've been using code from The Building Coder.
I'm fairly new to programming and totally new to C#, and I've never used any API before.

I also have other questions:

- Right now I'm typing the path and the name of the family item that I want to insert directly inside my code. Ideally, I'd like the user to pick the family when he starts the plugin, but I have no idea how to do that.
- Last thing I'd like to know is, after my family is inserted, how do I run automatically an interference check to see if there is any collision. The plugin is aimed to people totally new to geomatics, that have never used any software like AutoCAD, and I want them to push the least number of buttons possible to achieve their goal.

**Answer:**
Thank you for your query.

Looks like a very cool project.

Congratulations on getting so far already.

Let me address your question in four steps:

- [Dynamic face type checking](#2)
- [Arbitrary axis algorithm](#3)
- [Generic face tangent determination](#4)
- [Use UVPoint directly](#5)
- [Family path and name selection](#6)
- [Interference and collision checking](#7)

#### Dynamic Face Type Checking

You can easily check for the type of the face dynamically using something like this:

```csharp
  if( obj is PlanarFace )
  {
    PlanarFace planarFace = obj as PlanarFace;

    // Handle planar face case ...
  }
  else if( obj is CylindricalFace )
  {
    CylindricalFace cylindricalFace = obj
      as CylindricalFace;

    // Handle cylindrical face case ...
  }
```

It might be better not to separate between planar and cylindrical faces, though.

All you want is the vector v, is it?

And you want v to be tangential to the face surface?

That can be achieved in a generic fashion, independent of the face type.

#### Arbitrary Axis Algorithm

Actually, as far as I can tell, what you are after is just an arbitrary vector tangential to the face surface.

That reminds me of the age-old
[AutoCAD arbitrary axis algorithm](http://www.autodesk.com/techpubs/autocad/acadr14/dxf/arbitrary_axis_algorithm_al_u05_c.htm) and
the determination of a normal vector performed by the GetCurveNormal method:

- [Detail curve must be in plane](http://thebuildingcoder.typepad.com/blog/2010/05/detail-curve-must-be-in-plane.html)
- [Model curve creator](http://thebuildingcoder.typepad.com/blog/2010/05/model-curve-creator.html)
- [Plane normal and points on plane](http://thebuildingcoder.typepad.com/blog/2010/10/plane-normal-and-points-on-plane.html)

However, in this case, there is actually no need for all this, because the Revit API provides all we need built-in.

#### Generic Face Tangent Determination

Given a Revit API geometric face, the easiest way to determine the tangent vectors at a given point is provided by the generic Face.ComputeDerivatives method.

The only little twist required to make use of it is that it takes a UV point in the 2D face coordinate system as an input argument.
Given the 3D global XYZ point returned by the reference returned by PickObject, we can use the Face.Project method to determine the corresponding UV point on the face.

Thus, ComputeDerivatives can be used to determine a face tangent for any kind of face like this:

```csharp
  /// <summary>
  /// Place an instance of the given family symbol
  /// on a selected face of an existing 3D element.
  /// </summary>
  FamilyInstance PlaceFamilyInstanceOnFace(
    UIDocument uidoc,
    FamilySymbol symbol )
  {
    Document doc = uidoc.Document;

    Reference r = uidoc.Selection.PickObject(
      ObjectType.Face, "Please pick a point on "
      + " a face for family instance insertion");

    Element e = doc.GetElement( r.ElementId );

    GeometryObject obj
      = e.GetGeometryObjectFromReference( r );

    XYZ p = r.GlobalPoint;

    // Better than specialised individual handlers
    // for each specific case, handle the general
    // case in a generic fashion.

    Face face = obj as Face;
    IntersectionResult ir = face.Project( p );
    UV q = ir.UVPoint;
    Transform t = face.ComputeDerivatives( q );
    XYZ v = t.BasisX; // or BasisY, or whatever...

    return doc.Create.NewFamilyInstance(r, p, v, symbol);
  }
```

As you can see, this is elegant and generic – there is no need to care about the type of face at all, it works the same way for all, including face types that you may now even be aware of, e.g. conical, Hermite, revolved and ruled, and other unthinkable ones that might potentially be added in the future.

#### Use UVPoint Directly

Taking a second look at the possibilities offered, I notice that the PickPoint reference already provides direct access to the face UV point in case a surface has actually been selected, so there is no need for the projection of the 3D XYZ point and the code simplifies to this:

```csharp
  Reference r = uidoc.Selection.PickObject(
    ObjectType.Face, "Please pick a point on "
    + " a face for family instance insertion" );

  Element e = doc.GetElement( r.ElementId );

  GeometryObject obj
    = e.GetGeometryObjectFromReference( r );

  Face face = obj as Face;
  UV q = r.UVPoint;

  Transform t = face.ComputeDerivatives( q );
  XYZ v = t.BasisX; // or BasisY, or whatever...

  return doc.Create.NewFamilyInstance( r, p, v, symbol );
```

Your other questions are pretty interesting as well.

#### Family Path and Name Selection

To pick the path and name of the family to insert, you can use the .NET OpenFileDialog to select a family definition RFA file as demonstrated by the
[text file driven automatic placement of family instances](http://thebuildingcoder.typepad.com/blog/2013/10/text-file-driven-automatic-placement-of-family-instances.html) example
that uses it to
[select an input text file](http://thebuildingcoder.typepad.com/blog/2013/10/text-file-driven-automatic-placement-of-family-instances.html).
Once the family definition file is selected, you can load it with the LoadFamily method.
The Building Coder provides lots of examples of how to do that.

#### Interference and Collision Checking

Running an interference check is only a little bit more work, and there are probably various ways to go at it,
so I will just hint at one possible starting point here.
The first idea that comes to mind is the following:

- Determine the solid of the inserted family instance.
- Instantiate an ElementIntersectsSolidFilter using that.
- Retrieve all elements passing the filter.

[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
CmdCollectorPerformance external command is currently set up to call the GetInstancesIntersectingElement method to test and demonstrate use of the ElementIntersectsElementFilter, closely related to the ElementIntersectsSolidFilter, with a detailed analysis of the use and result provided by the discussion on
[determining all family instances intersecting an element](http://thebuildingcoder.typepad.com/blog/2014/11/determining-intersecting-elements-and-continued-futureproofing.html#3).

Furthermore, The Building Coder provides a whole topic group on various aspects of
[element intersection and collision detection](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.48).

I added all the code described above to the new method PlaceFamilyInstanceOnFace in the module
[CmdNewLightingFixture.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdNewLightingFixture.cs) in
[release 2015.0.117.3](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.117.3) of
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).

That should be more than enough to help you get on with the next steps of your add-in.
Good luck!

#### Autodesk Screencasts for Training Videos and Tutorials

If you are preparing any kind of training video or tutorial on how to achieve a specific computing task, check out the possibilities offered by
[Autodesk Screencasts](http://au.typepad.com/au/2015/02/now-featuring-drumroll-please-autodesk-screencasts.html):
a Screencast is like a screenshot in video form taking a ride-along with the person giving the presentation and seeing click-by-click how it is driven.