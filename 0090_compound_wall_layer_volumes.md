---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.1
content_type: code_example
optimization_date: '2025-12-11T11:44:13.343649'
original_url: https://thebuildingcoder.typepad.com/blog/0090_compound_wall_layer_volumes.html
post_number: 0090
reading_time_minutes: 7
series: general
slug: compound_wall_layer_volumes
source_file: 0090_compound_wall_layer_volumes.htm
tags:
- csharp
- elements
- geometry
- parameters
- python
- revit-api
- selection
- walls
title: Compound Wall Layer Volumes
word_count: 1365
---

### Compound Wall Layer Volumes

Here I continue the discussion of one of the topics that were raised in the
[Revit API training in Verona](http://thebuildingcoder.typepad.com/blog/2009/01/verona-revit-api-training.html)
last week.

We can combine the two discussion strands analysing
[wall areas](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html)
and
[compound wall layers](http://thebuildingcoder.typepad.com/blog/2008/11/wall-compound-layers.html)
to calculate the volumes of each individual layer in a compound wall.

In our initial idea for implementing this, we thought of making use of the wall area that we calculated ourselves from the appropriate face of the wall's solid geometry.
Whatever method we use to determine the area, there will be some imprecision in the calculation, since the wall profile area will be calculated either for the inner or outer face of the wall, which will not exactly match the area of some of the interior layers, depending on how the wall layers connect in the corners and wall intersections.

Later, we discovered that a simpler and sometimes exact approach is given by using the value of the built-in parameter HOST\_AREA\_COMPUTED instead.
In a compound wall with multiple layers, we can simply multiply this value with the thickness of each of the component layers to obtain the volume of each layer.
We implemented a test to compare the sum of all the component layer volumes with the total wall volume obtained from the HOST\_VOLUME\_COMPUTED parameter.
Depending on the type of wall connection, the two results are sometimes exactly equal.

Since the Revit database unit for length is feet, all our raw values will be in feet for length, square feet for area, and cubic feet for volume. We convert the data to metric before displaying it.

To store the cumulated volumes for each wall layer as well as the wall total, we make use of a dictionary mapping a key to a double number representing the volume. The key is defined by concatenating the wall type name and the layer function, using the string " : " as a separator.

Since we want to cumulate the volumes for all selected walls, we implemented a derived class MapLayerToVolume with a Cumulate method. If we used the generic Dictionary class directly, we would have to use the ContainsKey as well as the Add method to check whether the key is present before adding to its volume:

```csharp
class MapLayerToVolume
  : Dictionary<string, double>
{
  public void Cumulate(
    string key,
    double value )
  {
    if( !ContainsKey( key ) )
    {
      this[key] = 0.0;
    }
    this[key] += value;
  }
}
```

We define constants for the two parameters we are interested in and a little helper method to retrieve their values from a given wall:

```csharp
const BuiltInParameter \_bipArea
  = BuiltInParameter.HOST\_AREA\_COMPUTED;

const BuiltInParameter \_bipVolume
  = BuiltInParameter.HOST\_VOLUME\_COMPUTED;

double GetWallParameter(
  Wall wall,
  BuiltInParameter bip )
{
  Parameter p = wall.get\_Parameter( bip );

  Debug.Assert( null != p,
    "expected wall to have "
    + "HOST\_AREA\_COMPUTED and "
    + "HOST\_VOLUME\_COMPUTED parameters" );

  return p.AsDouble();
}
```

The method GetWallLayerVolumes uses this helper method to determine and cumulate the compound wall layer volumes for a given wall:

```python
void GetWallLayerVolumes(
  Wall wall,
  ref MapLayerToVolume totalVolumes )
{
  WallType wt = wall.WallType;

  CompoundStructure structure
    = wt.CompoundStructure;

  CompoundStructureLayerArray layers
    = structure.Layers;

  int i, n = layers.Size;
  double area = GetWallParameter( wall, \_bipArea );
  double volume = GetWallParameter( wall, \_bipVolume );
  double thickness = wt.Width;

  string desc = Util.ElementDescription( wall );

  Debug.WriteLine( string.Format(
    "{0} with thickness {1}"
    + " and volume {2}"
    + " has {3} layer{4}{5}",
    desc,
    Util.MmString( thickness ),
    Util.RealString( volume ),
    n, Util.PluralSuffix( n ),
    Util.DotOrColon( n ) ) );

  //
  // volume for entire wall:
  //
  string key = wall.WallType.Name;
  totalVolumes.Cumulate( key, volume );

  //
  // volume for compound wall layers:
  //
  if( 0 < n )
  {
    i = 0;
    double total = 0.0;
    double layerVolume;
    foreach( CompoundStructureLayer
      layer in layers )
    {
      key = wall.WallType.Name + " : "
        + layer.Function;

      layerVolume = area \* layer.Thickness;

      totalVolumes.Cumulate( key, layerVolume );
      total += layerVolume;

      Debug.WriteLine( string.Format(
        "  Layer {0}: function {1}, "
        + "thickness {2}, volume {3}",
        ++i, layer.Function,
        Util.MmString( layer.Thickness ),
        Util.RealString( layerVolume ) ) );
    }

    Debug.Print( "Wall volume = {0},"
      + " total layer volume = {1}",
      Util.RealString( volume ),
      Util.RealString( total ) );

    if( !Util.IsEqual( volume, total ) )
    {
      Debug.Print( "Wall host volume parameter"
        + " value differs from sum of all layer"
        + " volumes: {0}",
        volume - total );
    }
}
```

It iterates over each layer in the compound structure of the wall and determines its volume by multiplying the wall area with the layer thickness.
It checks whether the total wall volume obtained from the HOST\_VOLUME\_COMPUTED parameter is equal to the sum of all the individual layer volumes and reports the difference if this is not the case.

The mainline of the command selects the walls to process, applies GetWallLayerVolumes to each one in turn, and presents the results.
The user can preselect some specific walls, otherwise all walls in the project will be processed:

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;
List<Element> walls = new List<Element>();
if( !Util.GetSelectedElementsOrAll(
  walls, doc, typeof( Wall ) ) )
{
  Selection sel = doc.Selection;
  message = ( 0 < sel.Elements.Size )
    ? "Please select some wall elements."
    : "No wall elements found.";
  return CmdResult.Failed;
}

MapLayerToVolume totalVolumes
  = new MapLayerToVolume();

foreach( Wall wall in walls )
{
  GetWallLayerVolumes( wall, ref totalVolumes );
}

string msg
  = "Compound wall layer volumes formatted as '"
  + "wall type : layer function :";
  + " volume in cubic meters':\n";

List<string> keys = new List<string>(
  totalVolumes.Keys );

keys.Sort();

foreach( string key in keys )
{
  msg += "\n" + key + " : "
    + Util.RealString(
      Util.CubicFootToCubicMeter(
      totalVolumes[key] ) );
}

Util.InfoMsg( msg );

return CmdResult.Cancelled;
```

The command returns CmdResult.Cancelled, because no changes have been made to the Revit database.
If we return Succeeded, the document is unnecessarily marked as dirty.

Before displaying the volumes, they are converted from the raw internal cubic feet units to cubic meters using a new utility function CubicFootToCubicMeter:

```csharp
const double \_convertFootToMm = 12 \* 25.4;

const double \_convertFootToMeter
  = \_convertFootToMm \* 0.001;

const double \_convertCubicFootToCubicMeter
  = \_convertFootToMeter
  \* \_convertFootToMeter
  \* \_convertFootToMeter;

static public double CubicFootToCubicMeter(
double volume )
{
  return volume \* \_convertCubicFootToCubicMeter;
}
```

Here is the debug output produced by running this analysis in a simple project with just a few walls:

```
Walls <130328 Exterior - Block on Mtl. Stud> with thickness 460 mm and volume 679.68 has 7 layers:
  Layer 1: function Finish1, thickness 200 mm, volume 295.51
  Layer 2: function ThermalOrAir, thickness 76 mm, volume 112.29
  Layer 3: function MembraneLayer, thickness 0 mm, volume 0
  Layer 4: function Substrate, thickness 19 mm, volume 28.07
  Layer 5: function Structure, thickness 152 mm, volume 224.59
  Layer 6: function MembraneLayer, thickness 0 mm, volume 0
  Layer 7: function Finish2, thickness 13 mm, volume 19.21
Wall volume = 679.68, total layer volume = 679.68

Walls <130357 Exterior - Block on Mtl. Stud> with thickness 460 mm and volume 489.94 has 7 layers:
  Layer 1: function Finish1, thickness 200 mm, volume 213.02
  Layer 2: function ThermalOrAir, thickness 76 mm, volume 80.95
  Layer 3: function MembraneLayer, thickness 0 mm, volume 0
  Layer 4: function Substrate, thickness 19 mm, volume 20.24
  Layer 5: function Structure, thickness 152 mm, volume 161.89
  Layer 6: function MembraneLayer, thickness 0 mm, volume 0
  Layer 7: function Finish2, thickness 13 mm, volume 13.85
Wall volume = 489.94, total layer volume = 489.94

Walls <130424 Generic - 200mm> with thickness 200 mm and volume 791.05 has 1 layer:
  Layer 1: function Structure, thickness 200 mm, volume 791.05
Wall volume = 791.05, total layer volume = 791.05

Compound wall layer volumes formatted as 'wall type : layer function : volume in cubic meters':

Exterior - Block on Mtl. Stud : 33.12
Exterior - Block on Mtl. Stud : Finish1 : 14.4
Exterior - Block on Mtl. Stud : Finish2 : 0.94
Exterior - Block on Mtl. Stud : MembraneLayer : 0
Exterior - Block on Mtl. Stud : Structure : 10.94
Exterior - Block on Mtl. Stud : Substrate : 1.37
Exterior - Block on Mtl. Stud : ThermalOrAir : 5.47
Generic - 200mm : 22.4
Generic - 200mm : Structure : 22.4
```

To see the full text, you will have to copy and paste to a text editor.

The final result is displayed in a dialogue box:

![Compound wall layer volumes](img/compound_wall_layer_volumes.png)

Here is an updated
[version 1.0.0.21](http://thebuildingcoder.typepad.com/blog/files/bc10021.zip)
of the complete Visual Studio solution with this new command implementation.