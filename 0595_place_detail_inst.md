---
post_number: "0595"
title: "Placing a Line Based Detail Item Instance"
slug: "place_detail_inst"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'levels', 'parameters', 'references', 'revit-api', 'views']
source_file: "0595_place_detail_inst.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0595_place_detail_inst.html"
---

### Placing a Line Based Detail Item Instance

We already talked in depth about
[placing a detail family instance](http://thebuildingcoder.typepad.com/blog/2010/10/place-detail-instance.html) and
the difficulties in
[selecting the right overload of the NewFamilyInstance method](http://thebuildingcoder.typepad.com/blog/2011/01/newfamilyinstance-overloads.html) for
that, a topic also discussed in the developer guide section 12.3.5 Creating FamilyInstance Objects, and its tables 34 and 35.
As we already stated in that post, the exploration continues...

Here is an interesting update to the issue by Frode Tørresdal of
[Norconsult Informasjonssystemer](http://www.nois.no),
which also highlights a new overload added in the Revit 2012 API:

**Question:** I am trying to place a detail item over an existing 3D element (in this case it's a Rebar Bar family).
I try to place the detail item with the NewFamilyInstance method and then move it to the min value of the existing 3D element's boundary box.
The code looks something like this:
```csharp
  XYZ p = XYZ.Zero;
  FamilySymbol symbol = null;

  FamilyInstance newDetailItem = doc.Create
    .NewFamilyInstance( p, symbol, commandData.View );

  BoundingBoxXYZ boundingBox = rebar3D
    .get\_BoundingBox( commandData.View );

  newDetailItem.Location.Move( boundingBox.Min );
```

The detail item is line based.

What I'm trying to do is to make a function where the user selects a 3D reinforcement item.
Then my function hides this item and places a simplified 2D symbol over the extent of the 3D item in the active plan view.

I can do this manually in Revit by placing my detail item along one side of the 3D element to set the width, and then adjusting a parameter in the detail item to set the height.

Using the API, I can adjust the parameter to get the right height, but I cannot place the detail item correctly.
I might be missing a coordinate transformation because my item is placed in the same place no matter what 3D element I select in the view.

To summarize what I'm trying to do:

- Place a line based detail element at the min value of the bounding box of a selected 3D element in plan view.- Extend the line based element so that the length is equal to one of the sides of the bounding box of the selected 3D element in a plan view.

I hope this makes sense. I'm rather new to the Revit API and might be missing something obvious.

**Solution:** While creating a test case I think I solved the problem.

I found a new overload of the NewFamilyInstance method in Revit 2012 that takes Line, FamilySymbol and View input arguments.
This seems to work for my line based detail item.
Using the override with a point as the first parameter only works with point based detail items.

Many thanks to Frode for this discovery!

That leads me to look in more detail at the new overloads of the NewFamilyInstance method.

#### New overloads of the NewFamilyInstance method in the Revit 2012 API

The nine overloads of the NewFamilyInstance method provided by the Revit 2011 API take the following lists of input arguments:

- XYZ, FamilySymbol, StructuralType- XYZ, FamilySymbol, View- XYZ, FamilySymbol, Level, StructuralType- XYZ, FamilySymbol, Element, StructuralType- XYZ, FamilySymbol, Element, Level, StructuralType- XYZ, FamilySymbol, XYZ, Element, StructuralType- Curve, FamilySymbol, Level, StructuralType- Face, XYZ, XYZ, FamilySymbol- Face, Line, FamilySymbol

The Revit 2012 API provides twelve overloads:

- XYZ, FamilySymbol, StructuralType- XYZ, FamilySymbol, View- XYZ, FamilySymbol, Level, StructuralType- XYZ, FamilySymbol, Element, StructuralType- XYZ, FamilySymbol, Element, Level, StructuralType- XYZ, FamilySymbol, XYZ, Element, StructuralType- Curve, FamilySymbol, Level, StructuralType- Face, XYZ, XYZ, FamilySymbol- Face, Line, FamilySymbol- Line, FamilySymbol, View- Reference, XYZ, XYZ, FamilySymbol- Reference, Line, FamilySymbol

The three last ones are new, respectively intended for:

- Adding a line based detail family instance into the Revit document, using a line and a view where the instance should be placed.- Inserting a new instance of a family onto a face referenced by the input Reference instance, using a location, reference direction, and a type/symbol.- Inserting a new instance of a family onto a face referenced by the input Reference instance, using a line on that face for its position, and a type/symbol.

As the issues arising in this area indicate, the topic of adding a new family instance can be complex and sometimes requires a bit of research to find the right approach in each case.
I hope this helps a little.