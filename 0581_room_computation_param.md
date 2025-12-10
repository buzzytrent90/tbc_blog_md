---
post_number: "0581"
title: "New Room Computation Parameters"
slug: "room_computation_param"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'parameters', 'revit-api', 'rooms']
source_file: "0581_room_computation_param.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0581_room_computation_param.html"
---

### New Room Computation Parameters

Isidoro Aguilera of
[Aplicad](http://aplicad.com) submitted a
[comment](http://thebuildingcoder.typepad.com/blog/2011/04/migrating-the-building-coder-samples-to-revit-2012.html?cid=6a00e553e168978833014e8819fd5e970d#comment-6a00e553e168978833014e8819fd5e970d) asking
how to migrate code making use of some room computation parameters from the Revit 2011 API to Revit 2012, specifically the built-in parameters

- LEVEL\_ATTR\_ROOM\_COMPUTATION\_AUTOMATIC- LEVEL\_ATTR\_ROOM\_COMPUTATION\_HEIGHT

**Question:** I'm migrating a plug-in from 2011 and 2012 and experienced problems with the level type parameters BuiltInParameter.LEVEL\_ATTR\_ROOM\_COMPUTATION\_AUTOMATIC and BuiltInParameter.LEVEL\_ATTR\_ROOM\_COMPUTATION\_HEIGHT.
They were writable in Revit 2011, but are read-only in Revit 2012.

Have you experienced a similar issue when migrating your samples?

The following code worked in Revit 2011, and fails in 2012:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( App.Document );

  IList elementos = collector
    .OfCategory( BuiltInCategory.OST\_Levels )
    .WhereElementIsElementType()
    .ToElements();

  foreach( Element e in elementos )
  {
    Parameter p = e.get\_Parameter( BuiltInParameter
      .LEVEL\_ATTR\_ROOM\_COMPUTATION\_AUTOMATIC );

    p.Set( 0 );

    p = e.get\_Parameter( BuiltInParameter
      .LEVEL\_ATTR\_ROOM\_COMPUTATION\_HEIGHT );

    p.Set( 0 );
  }
```

**Answer:** In Revit 2012, levels no longer have the type parameters that existed in prior versions, and both the parameters you name are obsolete.
Instead of using LEVEL\_ATTR\_ROOM\_COMPUTATION\_HEIGHT, you can now use LEVEL\_ROOM\_COMPUTATION\_HEIGHT like this:
```csharp
      FilteredElementCollector collector
        = new FilteredElementCollector(
          uidoc.Document );

      IList<Element> elementos = collector
        .OfClass( typeof( Level ) )
        .ToElements();

      foreach( Element e in elementos )
      {
        Parameter p = e.get\_Parameter( BuiltInParameter
          .LEVEL\_ROOM\_COMPUTATION\_HEIGHT );

        p.Set( 2.5 );
      }
```

For setting the room height, you can use ROOM\_COMPUTATION\_HEIGHT.
The functionality around LEVEL\_ATTR\_ROOM\_COMPUTATION\_AUTOMATIC is totally unsupported now.