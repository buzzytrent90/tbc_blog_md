---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.0
content_type: qa
optimization_date: '2025-12-11T11:44:13.270860'
original_url: https://thebuildingcoder.typepad.com/blog/0038_wall_layers.html
post_number: 0038
reading_time_minutes: 7
series: general
slug: wall_layers
source_file: 0038_wall_layers.htm
tags:
- csharp
- elements
- geometry
- parameters
- revit-api
- selection
- views
- walls
- windows
title: Wall Compound Layers
word_count: 1320
---

### Wall Compound Layers

We have examined floor and slab geometry in some detail now, and would like to have a look at walls as well.
One important aspect of walls is their internal layering, defined by the compound layer structure.
In this post, we examine the layer geometry, i.e. how to determine the exact location of the different layers within the wall structure.
Many thanks to Harry Mattison for providing lots of support developing this code and explaining both the API and the user interface issues involved.

We have the following sources of input:

- Wall location curve and total wall thickness:

  ```
  LocationCurve curve = wall.Location as LocationCurve;
  double thickness = wall.WallType.Width;
  ```
- Compound structure layers and their individual thicknesses:

  ```
  CompoundStructure structure = wall.WallType.CompoundStructure;
  CompoundStructureLayerArray layers = structure.Layers;
  ```

How can these be used to determine the exact positions of the individual wall layers and the wall itself?

The most important bit of information in this context is that the wall centre line is always given by the wall location curve, and that is not affected by any other settings.

There are some additional wall parameters that we first thought might influence these positions:

- Location Line WALL\_KEY\_REF\_PARAM
- Location Line Offset WALL\_LOCATION\_LINE\_OFFSET\_PARAM

WALL\_KEY\_REF\_PARAM describes the setting of the wall's Location Line property. However, wall.Location is always the centre of the wall, not the position of the 'Location Line' visible in the user interface:

![Wall location line in user interface](img/wall_location_line_ui.png)

So this property will not affect the computation of the wall layer locations.

WALL\_LOCATION\_LINE\_OFFSET\_PARAM only affects walls that are used as panels in a curtain wall. It offsets the wall panel from the face of the curtain wall. Again, this parameter will not affect the wall layer analysis, because wall.Location will be the centre of the wall panel, regardless of any offset.

So now we have clarified the different inputs available, how can these be combined to determine the exact location of each layer?

From the wall and its location line, we can obtain the wall thickness and the wall centre line start and end point.
Assuming a horizontal wall base line and vertical sides, we can calculate the vectors v, pointing along the location line from its start to its end point, and w, pointing perpendicularly to the location line and the Z axis towards the exterior edge of the wall.
Both of these are normalised, i.e. one foot long, since the Revit database always uses feet as units for length measurements.
If we multiply them by the wall length and half the wall thickness and draw them starting from the location line start point, they look like this:

![Wall length and width vectors](img/wall_v_and_w.png)

We use the location line start and end points, the wall thickness, and these two vectors to construct various model lines:
the first simply displays the wall location line, extended by two feet in both directions beyond the end of the wall.
We draw a second model line representing w multiplied by half the wall thickness at the starting point of the first, perpendicular to it.
Then, we draw model lines deliminating the different wall layers.
We lengthen the layer delimination lines by one foot in both directions beyond the end of the wall.

Here is an example of a set of compound walls at various angles:

![Original compound walls](img/walls_compound.png)

Here is the result of running the wall layers command on that set, adding the model lines representing the wall location line, extending two feet beyond the end of each wall, its half thickness and exterior side, and the wall layer delimination lines extending one foot beyond the wall end:

![Compound walls with layer lines added](img/walls_compound_after.png)

We do need to pay attention to the wall Flipped property. If Flipped is true, the interior and exterior sides of the wall are swapped, so the layering is reversed. We can account for this reversal by negating w if Flipped is true.

Here is a more detailed view of one of the compound walls in its original state:

![Original compound wall](img/wall_compound.png)

Running the wall layers command adds the model lines described above:

![Compound wall with layer lines added](img/wall_compound_after.png)

Flipping the wall swaps it interior and exterior faces and reverses the layering:

![Flipped compound wall](img/wall_compound_flipped.png)

Checking the Flipped property and negating the vector w pointing to the exterior face reverses the order of the layer delimitation lines as well:

![Flipped compound wall with layer lines added](img/wall_compound_flipped_after.png)

While iterating over the wall layers, the command prints some data to the Visual Studio debug output window, listing the wall description, thickness, number of layers, and the function and thickness of each layer:

```
Walls <128581 Jeremy Compound> with thickness 300 mm has 4 layers:
  Layer 1: function Finish1, thickness 25 mm
  Layer 2: function ThermalOrAir, thickness 50 mm
  Layer 3: function Structure, thickness 200 mm
  Layer 4: function Finish2, thickness 25 mm
```

While preparing the snapshots above, I also had to deal with a completely different question, a simple user interface issue not related to the API: how can I display the detailed wall compound structure on the graphics screen and ensure that hatch patterns for the different layers are displayed? There are a few steps required: first, change the view property from Coarse to Fine. At that point, I see the lines dividing the layers, but no hatches. For the hatching, make sure the wall layers are defined to use materials that have a cut pattern. Also ensure that you are zoomed in enough in the view so the pattern does not overscale, i.e. turn to gray fill, because the lines are too close to each other.

Here is the CmdWallLayers source code:

```csharp
public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;
  List<RvtElement> walls = new List<RvtElement>();
  if( !Util.GetSelectedElementsOrAll(
    walls, doc, typeof( Wall ) ) )
  {
    Selection sel = doc.Selection;
    message = ( 0 < sel.Elements.Size )
      ? "Please select some wall elements."
      : "No wall elements found.";
    return CmdResult.Failed;
  }

  int i, n;
  double halfThickness, layerOffset;
  Creator creator = new Creator( app );
  XYZ lcstart, lcend, v, w, p, q;

  foreach( Wall wall in walls )
  {
    string desc = Util.ElementDescription( wall );

    LocationCurve curve
      = wall.Location as LocationCurve;

    if( null == curve )
    {
      message = desc + ": No wall curve found.";
      return CmdResult.Failed;
    }
    //
    // wall centre line and thickness:
    //
    lcstart = curve.Curve.get\_EndPoint( 0 );
    lcend = curve.Curve.get\_EndPoint( 1 );
    halfThickness = 0.5 \* wall.WallType.Width;
    v = lcend - lcstart;
    v = v.Normalized; // one foot long
    w = XYZ.BasisZ.Cross( v ).Normalized;
    if( wall.Flipped ) { w = -w; }

    p = lcstart - 2 \* v;
    q = lcend + 2 \* v;
    creator.CreateModelLine( p, q );

    q = p + halfThickness \* w;
    creator.CreateModelLine( p, q );

    // exterior edge
    p = lcstart - v + halfThickness \* w;
    q = lcend + v + halfThickness \* w;
    creator.CreateModelLine( p, q );

    CompoundStructure structure
      = wall.WallType.CompoundStructure;

    CompoundStructureLayerArray layers
      = structure.Layers;

    i = 0;
    n = layers.Size;
    Debug.WriteLine( string.Format(
      "{0} with thickness {1} mm"
      + " has {2} layer{3}{4}",
      desc,
      Util.FootToMm( 2 \* halfThickness ),
      n, Util.PluralSuffix( n ),
      Util.DotOrColon( n ) ) );

    if( 0 == n )
    {
      // interior edge
      p = lcstart - v - halfThickness \* w;
      q = lcend + v - halfThickness \* w;
      creator.CreateModelLine( p, q );
    }
    else
    {
      layerOffset = halfThickness;
      foreach( CompoundStructureLayer layer
        in layers )
      {
        Debug.WriteLine( string.Format(
          "  Layer {0}: function {1}, "
          + "thickness {2} mm",
          ++i, layer.Function,
          Util.FootToMm( layer.Thickness ) ) );

        layerOffset -= layer.Thickness;
        p = lcstart - v + layerOffset \* w;
        q = lcend + v + layerOffset \* w;
        creator.CreateModelLine( p, q );
      }
    }
  }
  return CmdResult.Succeeded;
}
```

I am adding version 1.0.0.13 of the complete Visual Studio solution
[here](http://thebuildingcoder.typepad.com/blog/files/bc10013.zip),
including the new CmdWallLayers and all other commands discussed so far. In fact, it even includes another command that we have not discussed yet, and will address in the next post.