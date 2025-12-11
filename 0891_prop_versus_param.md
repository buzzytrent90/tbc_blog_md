---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.1
content_type: code_example
optimization_date: '2025-12-11T11:44:14.806890'
original_url: https://thebuildingcoder.typepad.com/blog/0891_prop_versus_param.html
post_number: 0891
reading_time_minutes: 4
series: general
slug: prop_versus_param
source_file: 0891_prop_versus_param.htm
tags:
- csharp
- elements
- family
- filtering
- parameters
- python
- revit-api
- rooms
- views
- walls
title: Parameters versus Properties
word_count: 780
---

### Parameters versus Properties

In his
[comment](http://thebuildingcoder.typepad.com/blog/2013/01/read-material-asset-parameter.html?cid=6a00e553e168978833017ee80eb1cc970d#comment-6a00e553e168978833017ee80eb1cc970d) on
[reading material asset data](http://thebuildingcoder.typepad.com/blog/2013/01/read-material-asset-parameter.html),
[Alexander Buschmann](http://www.idat.de) points
out that this same information can be accessed without retrieving, reading and converting the data directly from the undocumented built-in parameters.

Instead and more comfortably, you can use the following official properties on the appropriate classes:
```csharp
  Material material = null;

  foreach( Material mat in e.Materials )
  {
    material = mat;
    break;
  }
  PropertySetElement pse = doc.GetElement(
    material.StructuralAssetId )
      as PropertySetElement;

  StructuralAsset asset = pse.GetStructuralAsset();

  double a = asset.WoodBendingStrength;
  double b = asset.WoodParallelCompressionStrength;
  double c = asset.WoodParallelShearStrength;
  double d = asset.WoodPerpendicularCompressionStrength;
  double f = asset.WoodPerpendicularShearStrength;
  // ... and lots of other properties ...
```

Even though a large and growing number of properties are available through this approach, the "Tension Parallel to Grain" one is actually currently not included in these, so you will indeed still have to use the direct parameter access anyway in this particular case.

Going through parameters is more generic, and most properties available on the classes are also available via parameters.

Parameters can also be used to define a parameter filter for a filtered element collector, so it is always useful to know that they are there and what they mean.

To discover the existence and use of specific parameters, you can use RevitLookup and
[BipChecker](http://thebuildingcoder.typepad.com/blog/2013/01/built-in-parameter-enumeration-duplicates-and-bipchecker-update.html) to
explore a simple model.
Modify the model interactively through the user interface and observe the effect on the parameters to discover their use and meaning.

He also adds:

For convenience, we use extension methods in our code to do this kind of thing, e.g.

```python
public static class ElementExtensions
{
  public static Material material( this Element e )
  {
    foreach( Material m in e.Materials )
    {
      return m;
    }
    return null;
  }
}
```

If this class part of an imported namespace, it can be called as:

```csharp
  Material material2 = e.material();
```

The compiler will translate that to this:

```csharp
  Material material3
    = ElementExtensions.material( e );
```

It is mostly just convenience, but it also improves the readability of the code, since the entire loop above collapses into just this one single property call.

Many thanks to Alexander for pointing this out!

Here are some pointers to other extension methods that we discussed in the past:

- [Scale a curve](http://thebuildingcoder.typepad.com/blog/2009/07/scale-a-curve.html)- [GetPolygon extension methods](http://thebuildingcoder.typepad.com/blog/2010/02/getpolygon-extension-methods.html)- [C# and .NET little wonders](http://thebuildingcoder.typepad.com/blog/2010/11/c-and-net-little-wonders.html)- [Top faces of sloped wall](http://thebuildingcoder.typepad.com/blog/2011/07/top-faces-of-wall.html)- [Unit conversion and display string formatting](http://thebuildingcoder.typepad.com/blog/2011/12/unit-conversion-and-display-string-formatting.html)- [Materials collection and filtering](http://thebuildingcoder.typepad.com/blog/2012/01/materials-collection-and-filtering.html)- [RevitRubyShell implementation and installer](http://thebuildingcoder.typepad.com/blog/2012/07/revitrubyshell-implementation-and-installer.html)- [Room in area predicate via point in polygon test](http://thebuildingcoder.typepad.com/blog/2012/08/room-in-area-predicate-via-point-in-polygon-test.html)- [FamilyParameter IsShared property](http://thebuildingcoder.typepad.com/blog/2012/09/familyparameter-isshared-property.html)

#### Show Viewport Extension Line or Label

Alexander also pointed out some other useful parameter to be aware of on the ViewType class in his
[comment](http://thebuildingcoder.typepad.com/blog/2013/01/changing-viewport-type.html?cid=6a00e553e168978833017d4046b782970c#comment-6a00e553e168978833017d4046b782970c) on
[changing the viewport type](http://thebuildingcoder.typepad.com/blog/2013/01/changing-viewport-type.html),
and my colleague Joe Ye just ran into a case dealing with the very same issue:

**Question:** We can change the length of the view title line:

![View title line](img/view_title_line_1.png)

For that we have to go to edit type and check the show extension line toggle:

![View title line check box](img/view_title_line_2.png)

It is possible to achieve this using the API?

**Answer:** Yes, you can set that check box via the API as well.
Here is the process:

1. Get the view type object.- Get the parameter that represents the "Show Extension Line".- Change the parameter value.

Here is an example of performing these three steps:

```csharp
  ElementId typeId = viewport.GetTypeId();

  ViewType viewtype = doc.GetElement( typeId )
    as ViewType;

  Parameter withLine = viewtype.get\_Parameter(
    BuiltInParameter
      .VIEWPORT\_ATTR\_SHOW\_EXTENSION\_LINE );

  withLine.Set( 0 ); // No Line
  withLine.Set( 1 ); // Has Line
```

Alexander also points out that the built-in parameter VIEWPORT\_ATTR\_SHOW\_LABEL affects the view label in a similar manner.