---
post_number: "0289"
title: "Insert Face-Hosted Sprinkler"
slug: "insert_sprinkler"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'references', 'revit-api', 'selection', 'walls']
source_file: "0289_insert_sprinkler.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0289_insert_sprinkler.html"
---

### Insert Face-Hosted Sprinkler

In between the series of background information from Scott's Autodesk University presentation on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html),
let's have a look at another example that combines a little bit of geometric analysis with other issues.
This example is from a case handled by Joe Ye and exploring how to insert a face-hosted sprinkler into the model.
A family instance is used to represent the sprinkler, so we presumably need to make use of one of the overloads of the NewFamilyInstance method.
The question is which one and how to supply the appropriate arguments.
We have looked at a number of related issues in the past:

- [Inserting a column](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-column.html).
- [Inserting a beam](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-beam.html).
- [Creating a slanted column](http://thebuildingcoder.typepad.com/blog/2009/06/creating-a-slanted-column.html).
- [Creating a curved beam](http://thebuildingcoder.typepad.com/blog/2009/06/creating-a-curved-beam.html).
- [Creating hosted family instances for lighting fixtures](http://thebuildingcoder.typepad.com/blog/2009/08/electrical-settings-and-lighting-fixtures.html#2).
- [Creating nested families](http://thebuildingcoder.typepad.com/blog/2009/11/nested-family.html).

The first two show how to check whether the required family is loaded and optionally load it if not.
The third discusses where to look in the developer guide for more information on the overloads of the NewFamilyInstance method.
The fifth has a certain similarity to this case, but still requires a different solution.
We can make use of all that information to tackle this current task as well, which requires yet a few more tricky details.

**Question:** I am having a problem inserting a face-hosted sprinkler symbol into my model.
I've been able to insert other family instances without problem, but this one isn't working for me.

I tried hosting the sprinkler on a ceiling object and on a face of the ceiling.
When trying to host the sprinkler on the ceiling, I tried creating the instance both with and without the ceiling level.
Every time it creates the sprinkler it places it at an elevation of zero even though my ceiling and insertion point is over 56'.
I also tried moving the elevation of the sprinkler after insertion but this did nothing.
Strangely, the XY location is correct.

I also tried to use a face of the ceiling to host the sprinkler.
This failed completely with an exception.
I tried using the face, the same insertion point, and a reference direction vector of (0, 0, -1) since I assume the sprinkler would point down the Z axis.

As an added note, I was unable to set the "Elevation" parameter of the Sprinkler after insertion.
It says that the operation cannot be done due to the state of the element.

**Answer:** To address the last note first, you mentioned that you cannot change the elevation parameter.
Yes, this is as designed, because this parameter is read-only and so cannot be modified.

Regarding the family instance creation, you need to use a specific overload of the NewFamilyInstance method to create new instance that is attached to the host face, namely:

```
FamilyInstance NewFamilyInstance(
  Face face,
  XYZ location,
  XYZ referenceDirection,
  FamilySymbol symbol
)
```

The face-hosted sprinkler requires a sketch plane, and all the other overloads of the NewFamilyInstance method cannot retrieve it together with the correct position and host element, so they end up creating an invalid instance without a correct sketch plane.

To place a sprinkler on the bottom of a ceiling element, we need to locate the ceiling's bottom face and create the sprinkler family instance on that face at a point located in it.
In addition, the face that we place the sprinkler on must be associated with a reference, or the placement will not work correctly.

I used the sample code implemented by Joe to create a new Building Coder external command CmdNewSprinkler demonstrating this.

In order to determine an insertion point for the sprinkler on the bottom face of the ceiling, it makes use of the following PointOnFace helper method.
It returns an arbitrary point on a planar face, namely the midpoint of the first mesh triangle encountered:
```csharp
XYZ PointOnFace( PlanarFace face )
{
  XYZ p = new XYZ( 0, 0, 0 );
  Mesh mesh = face.Triangulate();

  for( int i = 0; i < mesh.NumTriangles; ++i )
  {
    MeshTriangle triangle = mesh.get\_Triangle( i );
    p += triangle.get\_Vertex( 0 );
    p += triangle.get\_Vertex( 1 );
    p += triangle.get\_Vertex( 2 );
    p \*= 0.3333333333333333;
    break;
  }
  return p;
}
```

The Execute method of the CmdNewSprinkler command performs the following tasks:

- Check whether the required sprinkler family is loaded and load it if not.- Retrieve an arbitrary symbol from the sprinkler family, in this case, the first one encountered.- Prompt the user to select a ceiling element to host the sprinkler and ensure that a valid selection was made.- Retrieve the ceiling geometry, its solid, and determine the bottom face.- Use our PointOnFace helper method to determine an arbitrary insertion point on the ceiling bottom face.- Call the NewFamilyInstance method.

The trickiest step is the solid retrieval and the determination of the bottom face.
When retrieving the solid, we need to ensure that the options' ComputeReferences property is set, or the solid faces will not be associated with the required references.

In all previous Building Coder examples, we did not assign true to ComputeReferences, but left it in its default state of false, which presumably improves performance.
It would be interesting to test this, actually, and benchmark the difference that this option setting might have.
The setting is required in this case, because the NewFamilyInstance method needs the face reference to create the family instance and associate the sprinkler with the ceiling face.

To find the ceiling's bottom face, we assume that it is horizontal and flat, i.e. planar, so we can use its normal vector pointing straight down to identify it.

Here is the mainline code of the Execute method implementing this:
```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

// retrieve the sprinkler family symbol:

Filter filter = app.Create.Filter.NewFamilyFilter(
  \_name );

List<RvtElement> families = new List<RvtElement>();
doc.get\_Elements( filter, families );
Family family = null;

foreach( RvtElement e in families )
{
  family = e as Family;
  if( null != family )
    break;
}

if( null == family )
{
  if( !doc.LoadFamily( \_filename, out family ) )
  {
    message = "Unable to load '" + \_filename + "'.";
    return CmdResult.Failed;
  }
}

FamilySymbol sprinklerSymbol = null;
foreach( FamilySymbol fs in family.Symbols )
{
  sprinklerSymbol = fs;
  break;
}

Debug.Assert( null != sprinklerSymbol,
  "expected at least one sprinkler symbol"
  + " to be defined in family" );

// pick the host ceiling:

RvtElement ceiling = Util.SelectSingleElement(
  doc, "ceiling to host sprinkler" );

if( null == ceiling
  || !ceiling.Category.Id.Value.Equals(
    (int) BuiltInCategory.OST\_Ceilings ) )
{
  message = "No ceiling selected.";
  return CmdResult.Failed;
}

// retrieve the bottom face of the ceiling:

Options opt = app.Create.NewGeometryOptions();
opt.ComputeReferences = true;
GeoElement geo = ceiling.get\_Geometry( opt );

PlanarFace ceilingBottom = null;

foreach( GeometryObject obj in geo.Objects )
{
  Solid solid = obj as Solid;

  if( null != solid )
  {
    foreach( Face face in solid.Faces )
    {
      PlanarFace pf = face as PlanarFace;

      if( null != pf )
      {
        XYZ normal = pf.Normal.Normalized;

        if( Util.IsVertical( normal )
          && 0.0 > normal.Z )
        {
          ceilingBottom = pf;
          break;
        }
      }
    }
  }
}
if( null != ceilingBottom )
{
  XYZ p = PointOnFace( ceilingBottom );

  // create the sprinkler family instance

  FamilyInstance fi = doc.Create.NewFamilyInstance(
    ceilingBottom, p, XYZ.BasisX, sprinklerSymbol );

  return CmdResult.Succeeded;
}
return CmdResult.Failed;
```

To test the new command, we create a simple model with four walls and a ceiling:

![Ceiling element to host sprinkler](img/sprinkler_ceiling.png)

Running the command and selecting the ceiling element inserts the desired sprinkler:

![Newly created sprinkler family instance](img/sprinkler_instance.png)

Here is
[version 1.1.0.59](zip/bc11059.zip)
of the complete Building Coder sample source code and Visual Studio solution including the new command.

Many thanks to Joe for handling this case and creating the initial implementation!