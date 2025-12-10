---
post_number: "0278"
title: "Geometry Options"
slug: "abg01_geometry_options"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'levels', 'references', 'revit-api', 'rooms', 'views', 'walls', 'windows']
source_file: "0278_abg01_geometry_options.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0278_abg01_geometry_options.html"
---

### Geometry Options

This is part 1 of Scott Conover's AU 2009 class on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html).

It all starts with the get\_Geometry and its options.
What are these options?

Geometry is typically extracted from the indexed property Element.Geometry, or Element.get\_Geometry in C#.
This property accepts an options class which you must supply.
The options class customizes the type of output you receive:

- ComputeReferences – this flag sets Revit to make the GeometryObject.Reference property active for each geometry object extracted. When this is false, the property will be null. Remember that the default is false, you must turn this on or your reference will not be accessible- IncludeNonVisibleObjects – this flag new to Revit 2010 sets Revit to also return geometry objects which are not visible in a default view. Most of this non-visible geometry is construction and conditional geometry that the user sees when editing the element (for example, the center plane of a window family instance). So, typically you would set this to false (the default) as you don't want to include such information in your output. However, some of this conditionally visible geometry represents real-world objects, and in certain situations, you should extract it. One such example is the geometry of insulation surrounding ducts in Revit MEP.- DetailLevel – this sets the detail level for the extracted geometry. Note that the default for this is 'medium'.- View – this sets the view that governs the extracted geometry. Note that the detail level for this view will be used in place of 'DetailLevel' if a view is assigned (so you will need to change the view's detail level depending on your desired output). Set the view when you want to extract view-specific geometry not normally visible in the default 3D view.

#### Contents of the extracted geometry

The extracted geometry is returned to you as an Autodesk.Revit.Geometry.Element.
You can look at the geometry members of that element by iterating the Objects property.
Typically, the objects returned at the top level of the extracted geometry will be one of:

- Solid – a boundary representation made up of faces and edges.- Curve – a bounded 3D curve.- Instance – an instance of a geometric element, placed and positioned within the element.

An instance offers the ability to read its geometry through the GetSymbolGeometry and GetInstanceGeometry methods.
These methods return another Autodesk.Revit.Geometry.Element which can be parsed just like the first level return from get\_Geometry:

- GetSymbolGeometry returns the geometry represented in the coordinate system of the family. Use this, for example, when you want a picture of the 'generic' table without regards to the orientation and placement location within the project.- GetInstanceGeometry returns the geometry represented in the coordinate system of the project where the instance is placed. Use this, for example, when you want a picture of the specific geometry of the instance in the project (for example, ensuring that tables are placed parallel to the walls of the room).- There are also overloads for both methods that transform the geometry by any arbitrary coordinate system.

In some cases, instances may be nested many levels deep.
For example, a baluster in a railing placed in a document, or any other example of nested family instances.
We'll discuss transformations and their relationship to instance geometry later in this course.

#### Other sources of geometry

Geometry can also be obtained from a variety of other properties and methods:

- LocationCurve – curve driven elements like walls and beams report their profile through this interface.- References of dimensions – the references contain information regarding the geometry they point to.- Structural AnalyticalModel – returns curves and faces representing the analytical model of structural elements.- FindReferencesByDirection – discussed later in this course.