---
post_number: "0026"
title: "Family Instance Materials"
slug: "instance_materials"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'geometry', 'levels', 'references', 'revit-api', 'views', 'walls', 'windows']
source_file: "0026_instance_materials.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0026_instance_materials.html"
---

### Family Instance Materials

In the
[previous post](http://thebuildingcoder.typepad.com/blog/2008/10/element-materials.html),
we read out materials from walls and other building elements by querying them for their geometry,
picking out all the solids, and iterating over their faces. However, for family instances such as doors and windows, we noticed that no solid is present in the top level objects contained by the geometry element.

Family instances make use of a geometry instance to define their geometry. The geometry instance provides the facility to reuse symbol geometry in multiple locations for different family instances of the same symbol. Its most important properties to support this are:

- **Symbol:** The symbol element that this object is referring to.
- **SymbolGeometry:** The geometric representation of the symbol.
- **Transform:** The affine transformation from the local coordinate space of the symbol into the coordinate space of the instance.

In our current task of listing the materials used by the building elements, we do not need to examine the transformations applied to the instance geometry. If we were interested in face coordinates, we would have to keep track of that information.

In our case, to determine materials used in the door and window family instances, all we have to do is add one level of recursion to the existing GetMaterials() implementation. We need to check for geometry instance objects as well as solids. In case of an instance, we can access the symbol geometry and make a recursive call to GetMaterials() to pick out the materials from that as follows:

```csharp
public List<string> GetMaterials( GeoElement geo )
{
  List<string> materials = new List<string>();
  foreach( GeometryObject o in geo.Objects )
  {
    if( o is Solid )
    {
      Solid solid = o as Solid;
      foreach( Face face in solid.Faces )
      {
        string s = face.MaterialElement.Name;
        materials.Add( s );
      }
    }
    else if( o is GeoInstance )
    {
      GeoInstance i = o as GeoInstance;
      materials.AddRange( GetMaterials(
        i.SymbolGeometry ) );
    }
  }
  return materials;
}
```

Note that we added an additional alias to our namespace references to disambiguate the Revit element and geometry instance classes:

```
using GeoInstance = Autodesk.Revit.Geometry.Instance;
```

With the additional code in place, we obtain the same materials as before for the walls, floor and roof, and additionally a long list of new materials for the numerous family instance faces:

```
Walls <127248 Generic - 200mm> has 17 materials:
  0 Default Wall
  . . .
  16 Default Wall
Walls <127249 Generic - 200mm> has 6 materials:
  0 Default Wall
  . . .
  5 Default Wall
Walls <127250 Generic - 200mm> has 6 materials:
  0 Default Wall
  . . .
  5 Default Wall
Walls <127251 Generic - 200mm> has 6 materials:
  0 Default Wall
  . . .
  5 Default Wall
Doors M_Single-Flush <127252 0915 x 2134mm> has 26 materials:
  0 Door - Frame
  1 Door - Frame
  2 Door - Frame
  3 Door - Frame
  . . .
  19 Door - Frame
  20 Door - Panel
  21 Door - Panel
  . . .
  25 Door - Panel
Windows M_Fixed <127255 0406 x 0610mm> has 36 materials:
  0 Glass
  1 Glass
  . . .
  5 Glass
  6 Sash
  7 Sash
  . . .
  35 Sash
Windows M_Fixed <127258 0406 x 0610mm> has 36 materials:
  0 Glass
  1 Glass
  . . .
  5 Glass
  6 Sash
  7 Sash
  . . .
  35 Sash
Floors <127266 Generic Floor - 400mm> has 6 materials:
  0 Default Roof
  . . .
  5 Default Roof
Roofs <127269 Generic Roof - 300mm> has 12 materials:
  0 Default Roof
  . . .
  11 Default Roof
```

As you can see, the material assignments on the family instance solid faces may allow us to differentiate between the different element components, such as the door frame and panel, and the window glass and sash. This is however dependant on the family definition, so there is no guarantee that this approach will work for all families.

An interesting next step would be to analyse how to make use of the transformations defined by the nested geometry instances to calculate the real world coordinates. How to do this is actually demonstrated by the various
[geometry viewers](http://thebuildingcoder.typepad.com/blog/2008/09/geometry-viewer.html)
that we already discussed.

I am adding the complete Visual Studio solution [here](http://thebuildingcoder.typepad.com/blog/files/bc1006.zip). This version 1.0.0.6 includes all commands we discussed so far: CmdListWalls, CmdRelationshipInverter, CmdWallDimensions, CmdFilterPerformance and the updated version of CmdGetMaterials.