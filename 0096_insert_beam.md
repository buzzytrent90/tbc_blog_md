---
post_number: "0096"
title: "Inserting a Beam"
slug: "insert_beam"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'references', 'revit-api']
source_file: "0096_insert_beam.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0096_insert_beam.html"
---

### Inserting a Beam

This is the final Verona installment, discussing the last remaining topic raised in the
[Revit API training in Verona](http://thebuildingcoder.typepad.com/blog/2009/01/verona-revit-api-training.html)
the week before last, on creating new beam types and inserting beam instances through the API.

Similar to the exploration concerning
[columns](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-column.html),
we explored the following topics:

- Retrieving all matching family elements in current document in order to check whether the family we are interested in is loaded.
- Exploring the results of using a FamilyFilter.
- Loading a new family, if not already present.
- Creating a new family symbol, i.e. duplicating a beam type and setting its name and dimensions.
- Inserting a new beam instance into the model.

The interesting new aspects addressed with beams compared to columns are:

- The family type parameters and the naming convention used are completely different.
- Inserting a beam instance requires using a different version of NewFamilyInstance.

We implemented a new external command CmdNewBeamTypeInstance to demonstrate the steps listed above.
It is very similar to the command CmdNewColumnTypeInstance for the columns.
First, we set up some constants to define the family we are interested in, its library path, the structural type and a unit conversion:

```csharp
const string family\_name
  = "M\_Concrete-Rectangular Beam";

const string extension
  = ".rfa";

const string directory
  = "C:/Documents and Settings/All Users"
  + "/Application Data/Autodesk/RAC 2009"
  + "/Metric Library/Structural/Framing"
  + "/Concrete/";

const string path
  = directory + family\_name + extension;

StructuralType stBeam
  = StructuralType.Beam;
```

#### Creating a new Beam Type

Most of the implementation of the Execute method is the same as for
[CmdNewColumnTypeInstance](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-column.html).
We check whether the family we are interested in is already loaded, making use of a family filter to get all family elements in the current document. We again note that the family filter returns both the symbols contained within the family and the family itself. The family element itself is stored in the variable 'f', and its symbols are simply listed. In real life, we would probably eliminate the symbols by creating a Boolean 'and' filter and filtering for the Family class as well as the family name.
If the family was not already loaded, then 'f' remains null, and we load it with the LoadFamily method. It would also be sufficient to load one single symbol from the family, since we just need one single symbol 's' in order to call its Duplicate method. Any one will do, so we simply select the first one.
When duplicating it, we simultaneously define the new symbol name. We list all its parameters, set the new type's dimensions, and demonstrate that we can change its name at a later stage as well if desired:

```csharp
s.get\_Parameter( "b" ).Set(
  Util.MmToFoot( 500 ) );

s.get\_Parameter( "h" ).Set(
  Util.MmToFoot( 1000 ) );
s.Name = "Nuovo simbolo due";
```

The names of the dimension parameters we are interested in for the beam are named 'b' and 'h'.
The column class that we examined used a completely different naming convention, with full names and upper-case initial letters, e.g. 'Width' and 'Depth'.
This just goes to show that families are user defined and every family can use different conventions.

The new beam and column types appear like this in the Revit project browser:

![New column and beam types](img/new_beam_type.png)

To follow the standard Revit type naming conventions, we would normally include the new type dimensions in its name, like the existing types do.

#### Creating FamilyInstance Objects

We did some experiments in order to place an instance of our new symbol in the model.
First we verified that it is possible to insert a beam, which normally uses a location line, by specifying only a location point:

```csharp
XYZ p = XYZ.Zero;
doc.Create.NewFamilyInstance( p, s, nonStructural );
```

We can also place it with just a point and a direction:

```csharp
XYZ p = XYZ.Zero;
XYZ q = app.Create.NewXYZ( 30, 20, 20 ); // feet
FamilyInstance fi = doc.Create.NewFamilyInstance(
  p, s, q, null, nonStructural );
```

In both of these cases, the instance has no location line defined for it, causing it to behave rather strangely and basically be unusable, e.g. it cannot be selected.

It is also possible to place it on a level, but lacking a location line, this version is also not really useful:

```csharp
List<Element> levels = new List<Element>();
doc.get\_Elements( typeof( Level ), levels );
Debug.Assert( 0 < levels.Count,
  "expected at least one level in model" );

Level level = levels[0] as Level;

fi = doc.Create.NewFamilyInstance(
  line, s, level, nonStructural );
```

These various attempts led us to realise that we really do need to define a location line for the beam.
We also verified that we must specify a structural type. Specifying a non-structural type means that no beam is created, and results in a null family instance. So this is the final working version for inserting a valid instance of a beam with sloped location line:

```csharp
XYZ p = XYZ.Zero;
XYZ q = app.Create.NewXYZ( 30, 20, 20 ); // feet
Line line = app.Create.NewLineBound( p, q );
FamilyInstance fi = doc.Create.NewFamilyInstance(
  line, s, null, stBeam );
```

Here is an image showing the result of running the two new commands CmdNewBeamTypeInstance and CmdNewColumnTypeInstance, which insert one instance each of the of the new beam and column types:

![New column and beam instances](img/new_beam.png)

By the way, if you have any questions regarding the use of NewFamilyInstance and especially the choice of the correct overload to use for specific situations,
the first place to look is in the
[Revit 2009 API Developer Guide](http://thebuildingcoder.typepad.com/blog/files/Revit_2009_API_Developer_Guide.zip)
section 11.3.4 on Creating FamilyInstance Objects.
That document contains the most complete and up-to-date description of the topic.
That section was also provided temporarily as a stand-alone SDK document named
[Guide to placing Family Instances with the API.doc](C:/a/lib/revit/2009/SDK/Guide to placing Family Instances with the API.doc).
This document provides a roadmap on how to create different categories of family instances using the API.
Typically these instances are created using one of the eight overloads of the Autodesk.Revit.Creation.Document method NewFamilyInstance.
The choice of which overload to use depends not only on the category of the instance, but also other characteristics of the placement, such as whether it should be hosted, placed relative to a reference level, or placed directly on a particular face.
The details are included in a table.
Instances of some family types are better created through methods other than NewFamilyInstance and are listed in a second table.

Here is
[version 1.0.0.24](http://thebuildingcoder.typepad.com/blog/files/bc10024.zip)
of the complete Visual Studio solution with the new CmdNewBeamTypeInstance command implementation.