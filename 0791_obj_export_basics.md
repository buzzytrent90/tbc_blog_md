---
post_number: "0791"
title: "OBJ Model Export Considerations"
slug: "obj_export_basics"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'python', 'revit-api', 'selection', 'views', 'walls', 'windows']
source_file: "0791_obj_export_basics.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0791_obj_export_basics.html"
---

### OBJ Model Export Considerations

As I already said, I am
[getting going with the cloud](http://thebuildingcoder.typepad.com/blog/2012/06/getting-going-with-the-cloud.html),
and my first project is to export a Revit model to the
[Wavefront OBJ](http://en.wikipedia.org/wiki/Wavefront_.obj_file) file
format, which seems to be pretty standard and compact.

My exploration so far led to a number of observations and other issues:

- [Cloud and mobile DevBlog](#2)- [Selecting model elements](#3)- [OBJ file format](#4)- [Eliminating duplicate vertices](#5)- [XYZ vertex lookup](#6)- [Integer-based vertex lookup](#7)- [Next steps](#8)

#### Cloud and Mobile DevBlog

Talking about cloud and mobile, we have another addition to our growing list of ADN DevBlogs:

The
[Cloud and Mobile Development DevBlog](http://adndevblog.typepad.com/cloud_and_mobile) has now been launched.

For more background information on it and an overview of the other ADN DevBlogs, please refer to
[Kean's announcement](http://through-the-interface.typepad.com/through_the_interface/2012/06/cloud-mobile-devblog.html).

#### Selecting Model Elements

Before we can start exporting the Revit model geometry, we need to determine which elements to extract the data from.

We took a couple of stabs at this in the past; an early one was
[for Revit 2010](http://thebuildingcoder.typepad.com/blog/2009/05/selecting-model-elements.html)
([revisited](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html)),
selecting each element that:

- Is not a family or a family symbol- Has a valid category- Has non-empty geometry

This proved to be too simple, and
[further tests](http://thebuildingcoder.typepad.com/blog/2009/11/select-model-elements-2.html) were required to eliminate unwanted elements.

We revisited the topic using filtered element collectors to
[retrieve all model elements in Revit 2011](http://thebuildingcoder.typepad.com/blog/2010/10/selecting-model-elements.html)
([updated](http://thebuildingcoder.typepad.com/blog/2010/10/model-elements-revisited.html)).

Here are some other possible criteria to apply besides the three listed above:

- Category HasMaterialQuantities property- FilteredElementCollector WhereElementIsNotElementType and WhereElementIsViewIndependent methods

You may also find that you need to define detailed lists of categories or other criteria to satisfy for your specific requirements in selecting the elements to export.

#### OBJ File Format

Back to the OBJ format, one of its main advantages is its extremely succinct
[file format specification](http://www.eg-models.de/formats/Format_Obj.html).
As you can see, the only data that absolutely needs to be specified is the vertices and faces.
Optionally, texture coordinates and normals can also be defined.
The slightly more extensive
[Wikipedia version](http://en.wikipedia.org/wiki/Wavefront_.obj_file#File_format) also
explains how parameter space vertices can be specified, and how colours are defined using a material library.

Here is a sample OBJ file
[cube.obj](zip/wavefront_obj/cube.obj.txt) representing
a cube, with an associated material library
[material.mtl](zip/wavefront_obj/material.mtl.txt),
that I found in the
[.obj Loader for Android](http://sourceforge.net/projects/objloaderforand) distribution.

The cube definition is rather sub-optimal, though, listing and making use of 24 vertices, although we all know that a cube only has a total of eight corners.
Here is an optimised OBJ representation of a cube:

```
v 1 1 1
v 1 1 -1
v 1 -1 1
v 1 -1 -1
v -1 1 1
v -1 1 -1
v -1 -1 1
v -1 -1 -1
f 1 3 4 2
f 5 7 8 6
f 1 5 6 2
f 3 7 8 4
f 1 5 7 3
f 2 6 8 4
```

#### Eliminating Duplicate Vertices

As the trivial example above shows, you can significantly reduce the size of an OBJ by eliminating duplicate vertex definitions.
The generator of the larger cube sample file above defined 24 points, i.e. 4 \* 6.
It probably simply iterated over all six cube faces and output a new vertex definition for each face corner encountered, regardless of repetition.

This is very wasteful.
In Revit it also incurs the additional risk of defining vertices which are supposed to specify corners of adjacent faces, but may be slightly off, as we discussed in obtaining the
[top faces of sloped walls](http://thebuildingcoder.typepad.com/blog/2011/07/top-faces-of-wall.html),
[retrieving unique geometry vertices](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-day-two.html#2) from
a selected element, and most recently in the
[geometry traversal to retrieve unique vertices](http://thebuildingcoder.typepad.com/blog/2012/06/real-world-concrete-corner-coordinates.html#3) from
concrete structural elements.

If we collect all unique vertices in the Revit model and use the resulting list to avoid any duplicate definitions in the OBJ file, we can drastically reduce the number of vertex definitions, in this case from 24 to 8, and improve precision at the same time.

#### XYZ Vertex Lookup

According to the observations above, I implemented a vertex lookup class providing a method AddVertex taking an XYZ input argument specifying a new triangle vertex to store for exporting.

It looks up the given XYZ point and either returns the index of a previously defined point, if possible, or creates a new entry for it and returns that index.

As discussed so many times in the past, the point comparison has to include some fuzz.

Here is the full implementation of my VertexLookupXyz class with its own built-in XyzEqualityComparer:
```python
/// <summary>
/// A vertex lookup class to eliminate duplicate
/// vertex definitions
/// </summary>
class VertexLookupXyz : Dictionary<XYZ, int>
{
  #region XyzEqualityComparer
  /// <summary>
  /// Define equality for Revit XYZ points.
  /// Very rough tolerance, as used by Revit itself.
  /// </summary>
  class XyzEqualityComparer : IEqualityComparer<XYZ>
  {
    const double \_sixteenthInchInFeet
      = 1.0 / ( 16.0 \* 12.0 );

    public bool Equals( XYZ p, XYZ q )
    {
      return p.IsAlmostEqualTo( q,
        \_sixteenthInchInFeet );
    }

    public int GetHashCode( XYZ p )
    {
      return Util.PointString( p ).GetHashCode();
    }
  }
  #endregion // XyzEqualityComparer

  public VertexLookupXyz()
    : base( new XyzEqualityComparer() )
  {
  }
  /// <summary>
  /// Return the index of the given vertex,
  /// adding a new entry if required.
  /// </summary>
  public int AddVertex( XYZ p )
  {
    return ContainsKey( p )
      ? this[p]
      : this[p] = Count;
  }
}
```

The XYZ points that I consider here are in the original Revit model coordinate system, i.e. in feet.

#### Integer-based Vertex Lookup

Thinking about the issue a bit further, though, I discovered several enhancement possibilities.

On one hand, it might be more useful for me to create the OBJ files in millimetres instead of feet.

Secondly, if you don't require floating-point precision, using integers is always more efficient, and mostly simpler as well.

This led me to define my own integer-based point class for storing the vertex coordinates in millimetres instead of feet, and in whole numbers instead of floating point.

The rest of my algorithm remained completely unchanged, and the resulting OBJ file also only differed in the actual vertex coordinate values.

Here is the implementation of my VertexLookupInt class with its own built-in integer based point class PointInt and PointIntEqualityComparer:
```csharp
class PointInt : IComparable<PointInt>
{
  public int X { get; set; }
  public int Y { get; set; }
  public int Z { get; set; }

  const double \_feet\_to\_mm = 25.4 \* 12;

  static int ConvertFeetToMillimetres( double d )
  {
    return (int) ( \_feet\_to\_mm \* d + 0.5 );
  }

  public PointInt( XYZ p )
  {
    X = ConvertFeetToMillimetres( p.X );
    Y = ConvertFeetToMillimetres( p.Y );
    Z = ConvertFeetToMillimetres( p.Z );
  }

  public int CompareTo( PointInt a )
  {
    int d = X - a.X;

    if( 0 == d )
    {
      d = Y - a.Y;

      if( 0 == d )
      {
        d = Z - a.Z;
      }
    }
    return d;
  }
}

/// <summary>
/// A vertex lookup class to eliminate duplicate
/// vertex definitions
/// </summary>
class VertexLookupInt : Dictionary<PointInt, int>
{
  #region PointIntEqualityComparer
  /// <summary>
  /// Define equality for integer-based PointInt.
  /// </summary>
  class PointIntEqualityComparer : IEqualityComparer<PointInt>
  {
    public bool Equals( PointInt p, PointInt q )
    {
      return 0 == p.CompareTo( q );
    }

    public int GetHashCode( PointInt p )
    {
      return (p.X.ToString()
        + "," + p.Y.ToString()
        + "," + p.Z.ToString())
        .GetHashCode();
    }
  }
  #endregion // PointIntEqualityComparer

  public VertexLookupInt()
    : base( new PointIntEqualityComparer() )
  {
  }
  /// <summary>
  /// Return the index of the given vertex,
  /// adding a new entry if required.
  /// </summary>
  public int AddVertex( PointInt p )
  {
    return ContainsKey( p )
      ? this[p]
      : this[p] = Count;
  }
}
```

#### Next steps

I would dearly love to complete the entire discussion right here and now, but I guess that is enough for one go.

Here is an example of how far I have come with my implementation so far, exporting the rac\_advanced\_sample\_project.rvt to OBJ and viewing it in a Windows-based
[mesh viewer](http://mview.sourceforge.net),
still in mono-chrome, I'm afraid:

![Sample model OBJ file](img/obj_export_entire_model.png)

Here are some of the next steps I plan to take in the coming days:

- Discuss the rest of the OBJ export implementation in its current state- Add support for colour- Upload to the cloud- View on mobile device

It will be interesting to see where this leads us.
As always, feedback and suggestions are more than welcome.