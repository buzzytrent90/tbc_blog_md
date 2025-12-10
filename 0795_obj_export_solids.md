---
post_number: "0795"
title: "OBJ Model Exporter with Multiple Solid Support"
slug: "obj_export_solids"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'geometry', 'parameters', 'revit-api', 'selection', 'walls', 'windows']
source_file: "0795_obj_export_solids.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0795_obj_export_solids.html"
---

### OBJ Model Exporter with Multiple Solid Support

Yesterday, I thought that I had completed my basic
[working OBJ exporter](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-colours.html) implementation
including colour support.

Rudolf Honke of
[Mensch und Maschine acadGraph GmbH](http://www.acadgraph.de) rapidly
pointed out two important flaws, though.

Before discussing the ensuing OBJ exporter update, here is another little colour related item of Rudolf's:

#### Colour Conversion

Regarding colours, I have another hint.

When assigning a colour to a TextNoteType 'tnt', the required value is an integer.

It can be calculated as follows using a Windows colour input parameter, e.g. Color.Wheat:
```csharp
int GetRevitTextColorFromSystemColor(
  System.Drawing.Color color )
{
  return ( ( (int) color.R ) \* (int) Math.Pow( 2, 0 )
    + ( (int) color.G ) \* (int) Math.Pow( 2, 8 )
    + ( (int) color.B ) \* (int) Math.Pow( 2, 16 ) );
}
```

Here is an example of how it can be used:
```csharp
  // Set Revit text colour from system colour

  int color = GetRevitTextColorFromSystemColor(
    System.Drawing.Color.Wheat );

  tnt.get\_Parameter( BuiltInParameter.LINE\_COLOR )
    .Set( color );
```

This is of course a completely different issue than the OBJ exporter and its colour support, but for completeness sake...

#### Element with Multiple Solids

The first flaw reported by Rudolf was an error in the colour handling that was easy to fix.
I went in and updated the sample output image and source code zip file in yesterday's post with the corrected code right away.

Secondly, the basic sample model that I used for testing includes one interesting element that has caused issues in various geometry analysis scenarios several times in the past, the fireplace:
![Basic sample model fireplace](img/obj_export_fireplace.png)

My exporter simply grabs the first non-empty solid it can find, which happens to be the
[cuboid](http://en.wikipedia.org/wiki/Cuboid) mantelpiece.
All the rest of the fireplace is lost, including its chimney and even a section of the wall, leaving a hole in the outer shell of the house.

Exploring the fireplace element geometry in RevitLookup via Add-Ins > Revit Lookup > Snoop Current Selection... > FamilyInstance > Fireplace > Geometry > Objects > GeometryInstance > Symbol geometry > Objects, I see that it contains four different solids:
![Basic sample model fireplace solids](img/obj_export_fireplace_solids.png)

They have the following volumes, areas and number of faces:

- 220, 304, 11 planar- 15.8, 60, 6 planar + 8 cylindrical- 3.37, 32, 6 planar- 0, 0, 0

As always when dealing with lengths, areas and volumes in built-in
[Revit database units](http://thebuildingcoder.typepad.com/blog/units),
the area and volume is given in square and cubic feet, respectively.

Starting at the end, the fourth solid is empty and can be ignored.
The third is presumably the mantelpiece, the only element my initial exporter implementation handled.
The second is presumably the round chimney, and the first the actual fireplace itself, encompassing the burning chamber and missing wall section.

The exporter mainline that I presented last week was based on the idea of
[retrieving a maximum of one single solid per building element](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-exporter-take-one.html#7).
That assumption simply does not hold for this case.

#### Exporting Elements with Multiple Solids

I therefore replaced the
[GetSolid method](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-exporter-take-one.html#7) by
two new methods ExportSolids and ExportSolid, built to export any number of solids per element:
```csharp
bool ExportSolid(
  IJtFaceEmitter emitter,
  Document doc,
  Solid solid,
  Color color )
{
  foreach( Face face in solid.Faces )
  {
    Material m = doc.GetElement(
      face.MaterialElementId ) as Material;

    Color c = ( null == m ) ? color : m.Color;

    emitter.EmitFace( face,
      (null == c) ? \_default\_color : c );
  }
  return true;
}

/// <summary>
/// Export all non-empty solids found for
/// the given element. Family instances may have
/// their own non-empty solids, in which case
/// those are used, otherwise the symbol geometry.
/// The symbol geometry could keep track of the
/// instance transform to map it to the actual
/// project location. Instead, we ask for
/// transformed geometry to be returned, so the
/// resulting solids are already in place.
/// </summary>
int ExportSolids(
  IJtFaceEmitter emitter,
  Element e,
  Options opt,
  Color color )
{
  int nSolids = 0;

  GeometryElement geo = e.get\_Geometry( opt );

  Solid solid;

  if( null != geo )
  {
    Document doc = e.Document;

    if( e is FamilyInstance )
    {
      geo = geo.GetTransformed(
        Transform.Identity );
    }

    GeometryInstance inst = null;

    foreach( GeometryObject obj in geo )
    {
      solid = obj as Solid;

      if( null != solid
        && 0 < solid.Faces.Size
        && ExportSolid( emitter, doc, solid, color ) )
      {
        ++nSolids;
      }

      inst = obj as GeometryInstance;
    }

    if( 0 == nSolids && null != inst )
    {
      geo = inst.GetSymbolGeometry();

      foreach( GeometryObject obj in geo )
      {
        solid = obj as Solid;

        if( null != solid
          && 0 < solid.Faces.Size
        && ExportSolid( emitter, doc, solid, color ) )
        {
          ++nSolids;
        }
      }
    }
  }
  return nSolids;
}
```

With the new methods in place, the multiple solids of the single-element fireplace are properly handled.
The output element count reported now says:

![Fireplace exported element counts](img/obj_export_fireplace_msg.png)

The resulting OBJ output file is significantly more interesting and realistic:

![Fireplace OBJ model](img/obj_export_fireplace_obj.png)

Exporting the entire basic sample model obviously now also produces a different element count:

![Output element count with multiple solid support](img/obj_export_basic_model_solids_msg.png)

The difference to yesterday's counts is enormous, actually, indicating that there were several erroneous areas in the previous export, hopefully resolved by today's various improvements.

The model is looking better and better :-)

![OBJ output file with multiple solid support](img/obj_export_basic_model_solids_obj.png)

Well, actually, yes, a lot better, because we now see the windows, but also not better at all, because we can't see through them into the house any longer.

Apparently the windows were previously not being exported properly.
Probably just the window and door frames were exported, and not the glass.

Now the glass is included, and I have (intentionally) not yet done anything to support transparency, creating a dull result and making it impossible to look into the house.

The need to support transparency was actually one of the first things Rudolf pointed out in his
[comment](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html?cid=6a00e553e168978833017742c3ff3f970d#comment-6a00e553e168978833017742c3ff3f970d) on
the original
[exporter considerations](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html?cid=6a00e553e168978833017742c3ff3f970d#comment-6a00e553e168978833017742c3ff3f970d),
and now I see how right he was.

Well, here is
[ObjExport2.zip](zip/ObjExport2.zip) including
the entire source code, Visual Studio solution and add-in manifest for the updated OBJ exporter version 2 including support for multiple solids per element.

Many thanks to Rudolf for his many suggestions for improvements and prompting me to create this updated version.

I'll have a look at transparency next, and am still looking forward to the exploration of cloud and mobile support for this...