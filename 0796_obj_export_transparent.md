---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.610765'
original_url: https://thebuildingcoder.typepad.com/blog/0796_obj_export_transparent.html
post_number: 0796
reading_time_minutes: 6
series: general
slug: obj_export_transparent
source_file: 0796_obj_export_transparent.htm
tags:
- csharp
- elements
- references
- revit-api
- views
title: OBJ Model Exporter with Transparency Support
word_count: 1228
---

### OBJ Model Exporter with Transparency Support

Tuesday evening the moon was full, the rain finally eased up a bit, the sky became completely clear, and I spent a couple of hours outside on a hill with a beautiful view, a group of friends, and a nice hot fire.

![Full moon](file:////j/photo/jeremy/2012/2012-07-03_tuellinger_full_moon/img_1614_full_moon.jpg)

On Wednesday I finally got to play hockey again with my Autodesk colleagues after a long period of missing out on that.
I hope all US Americans enjoyed a nice
[Independence Day](http://en.wikipedia.org/wiki/Independence_Day_%28United_States%29).

The next days are filled with team meetings here in Neuchâtel, followed by our yearly Autodesk football tournament, taking place in Switzerland this year for the first time.
Unfortunately, the date for that was fixed very late, and I had already made a prior appointment for a mountain tour this coming weekend, so I will not be able to participate unless the weather turns really bad and prohibits mountaineering.
It's nice to have alternatives :-)

The OBJ model exporter I am working on went through an unexpected number of iterations from the
[base functionality](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html) through the
[first version](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-exporter-take-one.html),
then adding support for
[colour](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-colours.html) and processing
[multiple solids per BIM element](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-multiple-solid-support.html).

The last update forced me to realise that I really do need to support transparency as well, just as Rudolf Honke initially
[suggested](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-export-considerations.html?cid=6a00e553e168978833017742c3ff3f970d#comment-6a00e553e168978833017742c3ff3f970d).

#### Transparency in OBJ and Revit

Happily, the OBJ format and its associated
[MTL material libraries](http://people.sc.fsu.edu/~jburkardt/data/mtl/mtl.html)
([details](http://local.wasp.uwa.edu.au/~pbourke/dataformats/mtl))
do support this.

In MTL, the transparency is written using either a 'd' or a 'Tr' statement with values ranging from 0.0 to 1.0, where 1.0 is opaque.
This is the record generated for the glass in the basic sample model:

```
newmtl 4B475A69
Ka 0.27734375 0.3515625 0.41015625
Kd 0.27734375 0.3515625 0.41015625
d 0.25
```

Revit material transparency values lie between 0 and 100, where 100 is completely transparent and 0 opaque.
Here is the code that I am currently using to define a material library entry including transparency:
```csharp
  const string \_mtl\_newmtl\_d
    = "newmtl {0}\r\n"
    + "Ka {1} {2} {3}\r\n"
    + "Kd {1} {2} {3}\r\n"
    + "d {4}";

  s.WriteLine( \_mtl\_newmtl\_d,
    name,
    color.Red / 256.0,
    color.Green / 256.0,
    color.Blue / 256.0,
    (100 - transparency) / 100.0 );
```

Here is an impression of the resulting view in the
[Bonzai Engine](http://bonzaiengine.com) driven
[online model viewer](http://bonzaiengine.com/applet/modelviewer/modelviewer.php),
which was the only viewer I was able to find so far supporting transparency:

![Sample model with transparency](img/obj_export_basic_model_transparent.png)

#### Mingling Triangles, Colours and Transparency

The
[first take](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-exporter-take-one.html) of
the exporter used a simple list of integers representing triples of vertex indices to store the triangular facets to export.

When I decided to add
[colour support](http://thebuildingcoder.typepad.com/blog/2012/07/obj-model-exporter-with-colours.html) to
the exporter and realised that the mutual ordering of triangles and colour definitions for subsequent faces needs preserving, I decided to store the colours in the same list.

Since the Revit colour red, green and blue values are encoded in byte data, i.e. only range from 0 to 255, I can squeeze all three of them into a single integer like this:
```csharp
  static int ColorToInt( Color color )
  {
    return ( (int) color.Red ) << 16
      | ( (int) color.Green ) << 8
      | (int) color.Blue;
  }
```

For each colour switch, I create a triple of integers using a -1 marker, followed by one integer holding the colour value and a final zero-valued integer to keep up the multiples of three.

When adding support for transparency, I made use of the fact that the maximum transparency value returned by Revit is 100.
Left shifting that value by 24 bits results in 100 \* 2^24 = 1677721600, which is still small enough to fit into a signed integer, whose maximum value is 2^32 - 1 = 4294967295.
To put this shorter, you might also say that I make use of the fact that 100 is smaller than 127 = 2^7 - 1 :-)

I can thus encode transparency as well as colour in a single signed integer like this, including some assertions to ensure my thinking is not completely off track and Revit is sticking to its agreements:
```csharp
  public static int ColorTransparencyToInt(
    Color color,
    int transparency )
  {
    Debug.Assert( 0 <= transparency,
      "expected non-negative transparency" );

    Debug.Assert( 100 >= transparency,
      "expected transparency between 0 and 100" );

    uint trgb = ( (uint) transparency << 24 )
      | (uint) ColorToInt( color );

    Debug.Assert( int.MaxValue > trgb,
      "expected trgb smaller than max int" );

    return (int) trgb;
  }
```

These operations are reversed to extract the colour and transparency from the encoded integer like this:
```csharp
  static Color IntToColor( int rgb )
  {
    return new Color(
      (byte) ( ( rgb & 0xFF0000 ) >> 16 ),
      (byte) ( ( rgb & 0xFF00 ) >> 8 ),
      (byte) ( rgb & 0xFF ) );
  }

  public static Color IntToColorTransparency(
    int trgb,
    out int transparency )
  {
    transparency = (int) ( ( ( (uint) trgb )
      & 0xFF000000 ) >> 24 );

    return IntToColor( trgb );
  }
```

Since the materials referenced by the OBJ file need a name, I use a similar technique to generate that as well:
```csharp
  static string ColorString( Color color )
  {
    return color.Red.ToString( "X2" )
      + color.Green.ToString( "X2" )
      + color.Blue.ToString( "X2" );
  }

  public static string ColorTransparencyString(
    Color color,
    int transparency )
  {
    return transparency.ToString( "X2" )
      + ColorString( color );
  }
```

To find out what other exciting things happen to the data on its way out of the Revit model into the OBJ file, please have a look at the source code yourself.

Here is
[ObjExport3.zip](zip/ObjExport3.zip) including
the entire source code, Visual Studio solution and add-in manifest for the updated OBJ exporter version 3 including transparency support.

As far as I can tell, that concludes the OBJ export effort for now.
Next steps may include looking into viewing this on a mobile device and getting it uploaded to the cloud to make it available there.
I did define an abstract interface for the exporter, albeit very simple, in order to be able to replace the implementation to address other targets.
The future will tell where this will lead us.

#### Getting Started with the Revit 2013 API

Most of the suggestions provided for
[getting started with the Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2011/10/getting-started-with-the-revit-2012-api.html) and
[preparing for a hands on Revit API training](http://thebuildingcoder.typepad.com/blog/2012/01/preparing-for-a-hands-on-revit-api-training.html) are
still valid for Revit 2013 as well.

Still, for the sake of convenience and completeness, Mikako Harada updated the various pointers to the available materials and published an overview for
[getting started with the Revit 2013 API](http://adndevblog.typepad.com/aec/2012/07/getting-started-with-revit-2013-api.html) that
is well worth taking a gander at.