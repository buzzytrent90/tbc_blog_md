---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:14.117529'
original_url: https://thebuildingcoder.typepad.com/blog/0532_system_family_creation.html
post_number: '0532'
reading_time_minutes: 2
series: family
slug: system_family_creation
source_file: 0532_system_family_creation.htm
tags:
- csharp
- elements
- family
- revit-api
- walls
title: System Family Creation
word_count: 442
---

### System Family Creation

Here is a frequent query that came in once again today:

**Question:** I would like to create new system families in the Revit project, specifically wall families, to represent my stored database values including data such as material, thickness and other required properties.

I would like to use the API to create a new wall and assign the structural layers their function and material.
I would appreciate some advice how to achieve the creation of these system family types.

**Answer:** System families are built into Revit itself and cannot be modified, so the short answer to your question is simply 'No, you cannot create new system families'.

On the other hand, just like standard families, the purpose of system families is to provide types or symbols for use in the building model.
The types are represented by the ElementType base class, and it provides a Duplicate method which allows you to create new types of the same family.

Some system families are exposed as classes to the API, such as Wall.
In that case, the Wall class and its properties and methods define what you can do with it, and the wall types are represented by the WallType derived from ElementType.

Here is some VB code taken from the discussion on
[creating a new type](http://thebuildingcoder.typepad.com/blog/2008/11/creating-a-new-family-symbol.html) which
creates a new wall type using the Duplicate method, modifies it, and then assigns it to an existing wall:
```csharp
  Dim wallType As WallType = wall.WallType

  Dim newWallType As WallType \_
    = TryCast(wallType.Duplicate( \_
      newWallTypeName), WallType)

  Dim layers As CompoundStructureLayerArray \_
    = newWallType.CompoundStructure.Layers

  For Each layer As CompoundStructureLayer \_
    In layers

    ' double each layer thickness:

    layer.Thickness \*= 2.0R
  Next

  ' assign the new wall type back to the wall:

  wall.WallType = newWallType
```

For completeness sake, here is the same code in C#:
```csharp
  WallType wallType = wall.WallType;

  WallType newWallType = wallType.Duplicate(
    newWallTypeName ) as WallType;

  CompoundStructureLayerArray layers
    = newWallType.CompoundStructure.Layers;

  foreach( CompoundStructureLayer layer in layers )
  {
    // double each layer thickness:

    layer.Thickness \*= 2.0;
  }
  // assign the new wall type back to the wall:

  wall.WallType = newWallType;
```

In the current Revit API, you cannot create new layers in a wall type's compound layer structure.

The work-around for this is to create the required number of layers manually and save separate wall types for each number of layers in the template file used to create your project.
Then, when your application needs a new wall type with a specific number of layers, it can use a suitable one of the manually created predefined wall types and duplicate and modify that.