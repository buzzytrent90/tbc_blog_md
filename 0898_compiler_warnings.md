---
post_number: "0898"
title: "Eliminating Compiler Warnings and Deprecated Calls"
slug: "compiler_warnings"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'levels', 'revit-api', 'rooms', 'views', 'walls']
source_file: "0898_compiler_warnings.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0898_compiler_warnings.html"
---

### Eliminating Compiler Warnings and Deprecated Calls

I am revisiting this topic again, which we last looked at a year ago, to clean up our usage of
[deprecated methods in the Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2012/01/eliminating-compiler-warnings-and-deprecated-calls.html).

In that discussion, I also pointed back to previous versions and other related issues.

Here are the warnings about obsolete usage currently produced when compiling The Building Coder samples
[version 2013.0.100.1](zip/bc_13_100_1.zip) against the Revit 2013 API – copy to a text editor or view source to see the truncated lines in full:

```
C:\bc\BuildingCoder\BuildingCoder\CmdTransformedCoords.cs(67,41): warning CS0618: 'Autodesk.Revit.DB.GeometryElement.Objects' is obsolete: 'This property will be obsolete from 2013; Call GetEnumerator() instead.'
C:\bc\BuildingCoder\BuildingCoder\CmdNestedInstanceGeo.cs(119,32): warning CS0618: 'Autodesk.Revit.DB.GeometryElement.Objects' is obsolete: 'This property will be obsolete from 2013; Call GetEnumerator() instead.'
C:\bc\BuildingCoder\BuildingCoder\CmdLinkedFiles.cs(103,41): warning CS0618: 'Autodesk.Revit.DB.GeometryElement.Objects' is obsolete: 'This property will be obsolete from 2013; Call GetEnumerator() instead.'
C:\bc\BuildingCoder\BuildingCoder\CmdSetTagType.cs(171,19): warning CS0618: 'Autodesk.Revit.Creation.Document.NewWall(Autodesk.Revit.DB.Curve, Autodesk.Revit.DB.Level, bool)' is obsolete: 'This method is obsolete in Revit 2013. Please call a static creation method of Wall class instead.'
C:\bc\BuildingCoder\BuildingCoder\CmdCreateGableWall.cs(89,19): warning CS0618: 'Autodesk.Revit.Creation.Document.NewWall(Autodesk.Revit.DB.CurveArray, Autodesk.Revit.DB.WallType, Autodesk.Revit.DB.Level, bool, Autodesk.Revit.DB.XYZ)' is obsolete: 'This method is obsolete in Revit 2013. Please call a static creation method of Wall class instead.'
C:\bc\BuildingCoder\BuildingCoder\CmdPlanTopology.cs(103,26): warning CS0618: 'Autodesk.Revit.DB.PlanTopology.Rooms' is obsolete: 'This property is obsolete in Revit 2013.  Call GetRoomIds() instead.'
C:\bc\BuildingCoder\BuildingCoder\CmdPressKeys.cs(222,19): warning CS0618: 'Autodesk.Revit.Creation.Document.NewWall(Autodesk.Revit.DB.Curve, Autodesk.Revit.DB.WallType, Autodesk.Revit.DB.Level, double, double, bool, bool)' is obsolete: 'This method is obsolete in Revit 2013. Please call a static creation method of Wall class instead.'
C:\bc\BuildingCoder\BuildingCoder\CmdSlopedWall.cs(65,19): warning CS0618: 'Autodesk.Revit.Creation.Document.NewWall(Autodesk.Revit.DB.CurveArray, bool)' is obsolete: 'This method is obsolete in Revit 2013. Please call a static creation method of Wall class instead.'
C:\bc\BuildingCoder\BuildingCoder\CmdSlopedWall.cs(140,19): warning CS0618: 'Autodesk.Revit.Creation.Document.NewWall(Autodesk.Revit.DB.CurveArray, Autodesk.Revit.DB.WallType, Autodesk.Revit.DB.Level, bool, Autodesk.Revit.DB.XYZ)' is obsolete: 'This method is obsolete in Revit 2013. Please call a static creation method of Wall class instead.'

Compile complete -- 0 errors, 9 warnings
```

Basically, there are just three different warnings:

- The GeometryElement.Objects property is obsolete; [GeometryElement itself is now enumerable](#2), so you can iterate directly over the instance instead of querying its obsolete property.- The creation document NewWall method is obsolete, and replaced by [static creation methods on the Wall class](#3).- The PlanTopology.Rooms property is obsolete, and can be replaced using the [GetRoomIds method](#4).

Let's tackle those right here and now to future-proof this guy.

#### Enumerable GeometryElement

The first one is caused by accessing the GeometryElement.Objects property like this:

```csharp
  GeometryObjectArray objects = geoElem.Objects;

  n = objects.Size;

  // . . .

  foreach( GeometryObject obj in objects )
```

The GeometryElement is now itself iterable, so the foreach loop can be implemented on geoElem itself directly, eliminating the need for the intermediate objects variable.

The number of geometry objects is only used in an informational message and can be determine through the generic templated LINQ Count method:

```csharp
  n = geoElem.Count<GeometryObject>();

  // . . .

  foreach( GeometryObject obj in geoElem )
```

The next two warnings are eliminated in the same way.

#### Static Creation Methods on Wall Class

We then get to the call using the creation document NewWall method:

```csharp
  Wall wall = createDoc.NewWall(
    line, levelBottom, false );
```

This is simply replaced by a call to the new Wall class static Create method taking the same arguments plus the document:

```csharp
  Wall wall = Wall.Create(
    doc, line, levelBottom.Id, false );
```

The second occurence of this method in CmdCreateGableWall defines a profile for the wall.
It was passed in using a CurveArray instance in 2012:
```csharp
  // Create wall profile

  CurveArray profile = new CurveArray();

  XYZ q = pts[pts.Length - 1];

  foreach( XYZ p in pts )
  {
    profile.Append( appCreation.NewLineBound(
      q, p ) );

    q = p;
  }

  // . . .

  Wall wall = doc.Create.NewWall(
    profile, wallType, level, true, normal );
```

The Revit 2013 API eliminates the need for the custom collection class and uses a standard .NET List instead:

```csharp
  List<Curve> profile = new List<Curve>(
    pts.Length );

  XYZ q = pts[pts.Length - 1];

  foreach( XYZ p in pts )
  {
    profile.Add( appCreation.NewLineBound(
      q, p ) );
    q = p;
  }

  // . . .

  Wall wall = Wall.Create(
    doc, profile, wallType.Id, level.Id, true, normal );
```

#### PlanTopology GetRoomIds Method

The PlanTopology Rooms property is obsolete and should be replaced by GetRoomIds, e.g. in the following old code in CmdPlanTopology:

```csharp
  foreach( Room r in pt.Rooms )
  {
    output += "\n  " + r.Name + " : "
      + Util.RealString( r.Area ) + " sqf";
  }
```

Here is a possible reimplementation of that:

```csharp
  foreach( ElementId id in pt.GetRoomIds() )
  {
    Room r = doc.GetElement( id ) as Room;

    output += "\n  " + r.Name + " : "
      + Util.RealString( r.Area ) + " sqf";
  }
```

The rest of the warnings are pointing out more calls to the obsolete NewWall method, that I converted similarly to above.

Here is
[version 2013.0.100.2](zip/bc_13_100_2.zip) of
The Building Coder samples all deprecated API calls eliminated.

It also includes Victor's little
[update to the room retrieval](http://thebuildingcoder.typepad.com/blog/2011/11/accessing-room-data.html?cid=6a00e553e168978833017c3690489f970b#comment-6a00e553e168978833017c3690489f970b) sample.

All obsolete lines are marked with comments saying '// 2012', and their replacements are marked '// 2013'.
If you wish to analyse the exact differences, you can simply compare the code of this version with the
[version 2013.0.100.1](http://thebuildingcoder.typepad.com/files/bc_13_100_1-1.zip) implementing
the updated
[CmdDemoCheck command and serial number detection](http://thebuildingcoder.typepad.com/blog/2013/01/determine-revit-demo-mode-and-serial-number.html).