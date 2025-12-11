---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.8
content_type: code_example
optimization_date: '2025-12-11T11:44:13.416880'
original_url: https://thebuildingcoder.typepad.com/blog/0134_nested_instance_geo.html
post_number: '0134'
reading_time_minutes: 11
series: elements
slug: nested_instance_geo
source_file: 0134_nested_instance_geo.htm
tags:
- csharp
- elements
- family
- geometry
- levels
- python
- revit-api
- selection
- views
- windows
title: Nested Instance Geometry
word_count: 2134
---

### Nested Instance Geometry

This is in response to a
[query from Benson](http://thebuildingcoder.typepad.com/blog/2008/09/geometry-viewer.html?cid=6a00e553e1689788330115705bbc01970b#comment-6a00e553e1689788330115705bbc01970b)
about the structure and geometry of nested family instances.
In the answer, we also present some useful comparison operators which enable us to use Revit XYZ points and vectors as keys in generic .NET dictionaries, and sort them lexicographically, as well as utility functions for extracting and listing all the vertices of a solid.
Here is the original query, slightly rephrased:

**Question:**
I would like to obtain the internal hierarchy defined within a family instance, but its geometry only returns a flat list of solids.

For example: a top level family instance F includes one column C0, and two nested family instances F1 and F2.
F1 includes two more columns C1 and C2, and F2 includes two further columns C3 and C4.
The result I am trying to achieve is the geometry with the associated hierarchy:

```
  F
  }
  +-- C0
  }
  +-- F1
  }   }
  }   +-- C1
  }   }
  }   +-- C2
  }
  +-- F2
      }
      +-- C3
      }
      +-- C4
```

Here are my steps to create the family instances in this structure:

1. Run Revit 2009.- Select File > New > Family and open a metric template such as Metric Funiture.rft.- Delete all objects in the new family project, switch to 3D view.- Select File > Load from Library > Load Family and select an RFA file, with a column in it.- Now, we can find the column in the project browser panel > Families > Columns.- Select this column, right click on it, and select "Create Instance".
             Repeat step 6, so we now have two columns in the family project.- Save the project as myFamily1.rfa, then click File > Save As to save it again as myFamily2.rfa.
               Now, we have two family projects myFamily1.rfa and myFamily2.rfa.- Repeat steps 2, 3, and 4, this time selecting a column family, myFamily1, and myFamily2.- Create instances of the column, myFamily1, myFamily2 in the current family project like step 6.- Save the project as myFamily3.rfa.- Create a normal project, for instance named Project1.rvt, using File > New > Project.- Load myFamily3.rfa like step 4.- Add an instance of myFamily3 like step 6.

Now we have the structure shown above, with F represented by myFamily3, and F1 and F2 by myFamily1 and myFamily2 respectively.

**Answer:**
The geometry of the top level family instance F does indeed just contain a flat list of solids.
You can obtain some information about the hierarchical structure of the nested family instances by recursively exploring the family instance symbol family components.
For a given family instance a, they are accessible through a.Symbol.Family.Components.
These components include information about their location within the containing family.
Using this information, you can build up the structure you seek.

I created the project as you suggest and analysed the geometry and symbol family components of the
top level family instance F.
I used a rectangular column, so each of the columns is a simple quadrilateral with six faces.

In the geometry of the top level family instance F, I see one instance object.
The important properties of a geometry instance are its Symbol, SymbolGeometry, and Transform:

- Symbol, the symbol element that this object is referring to.- SymbolGeometry, the geometric representation of the symbol.- Transform, the affine transformation of the symbol into the instance coordinate space.

In my case, the symbol geometry contains six solids.
One of these is empty, i.e. has no edges or faces, so I simply ignore that one.
The remaining five represent the five different column instances visible in the project.
These five different instances can be differentiated by the different face vertex coordinates.

To explore and compare complex geometry like that of a solid, it is important to have some powerful tools at your disposal to reduce the amount of detailed information you have to analyse and compare.
In this case, in order to display the solids in an easily read fashion, I implemented a helper method named GetVertices to extract the vertices of a solid and store them in a sorted list in a XYZArray.

GetVertices iterates over all the faces of the given solid, triangulates each face, and iterates over the vertices of the resulting mesh.
These are added into a container, which is designed to ensure that every vertex occurs once only.
I implement this container by using a generic dictionary class and attaching an equality comparison class which ensures that almost equal points are regarded as being exactly equal, allowing for some fuzziness due to numerical imprecision.
Here is the definition of my XyzEqualityComparer class:

```python
class XyzEqualityComparer : IEqualityComparer<XYZ>
{
  public bool Equals( XYZ p, XYZ q )
  {
    return p.AlmostEqual( q );
  }

  public int GetHashCode( XYZ p )
  {
    return Util.PointString( p ).GetHashCode();
  }
}
```

The hash code calculation could certainly be optimised by avoiding the detour through a string representation, but this is the quickest and easiest solution that came to mind.
I am not sure that it will work reliably under all circumstances, though.
It is probably necessary to increase the precision in the string representation generated by PointString to match the precision used by the AlmostEqual method.

With this equality comparer, we can employ the standard .NET generic dictionary class using Revit XYZ points and vectors as dictionary keys.

In order to further facilitate the comparison of solids with each other, I sort their vertices before storing them into the array.
The sorting process requires another fuzzy comparison routine, a Compare function which determines whether a given point is larger than, equal to, or smaller than another one, and correspondingly returns 1, 0 or -1.
We define this comparison algorithm by comparing the X, Y and Z coordinates in the canonical order.
If all three are almost equal, the two points are considered equal.
Otherwise, the first pair of coordinates to differ is used to determine whether the first point is considered to be greater or smaller than the second.
Here is the implementation of this comparison method:

```csharp
const double \_eps = 1.0e-9;

public static bool IsZero( double a )
{
  return \_eps > Math.Abs( a );
}

public static bool IsEqual( double a, double b )
{
  return IsZero( b - a );
}

public static int Compare( double a, double b )
{
  return IsEqual( a, b ) ? 0 : ( a < b ? -1 : 1 );
}

public static int Compare( XYZ p, XYZ q )
{
  int diff = Compare( p.X, q.X );
  if( 0 == diff ) {
    diff = Compare( p.Y, q.Y );
    if( 0 == diff ) {
      diff = Compare( p.Z, q.Z );
    }
  }
  return diff;
}
```

With these two comparison helpers in place, I can implement my GetVertices helper method to extract a sorted list of solid vertices and store them in a sorted manner in a XYZArray like this:

```csharp
static void GetVertices( XYZArray vertices, Solid s )
{
  Debug.Assert( 0 < s.Edges.Size,
    "expected a non-empty solid" );

  Dictionary<XYZ, int> a
    = new Dictionary<XYZ, int>(
      new XyzEqualityComparer() );

  foreach( Face f in s.Faces )
  {
    Mesh m = f.Triangulate();
    foreach( XYZ p in m.Vertices )
    {
      if( !a.ContainsKey( p ) )
      {
        a.Add( p, 1 );
      }
      else
      {
        ±±a[p];
      }
    }
  }
  List<XYZ> keys = new List<XYZ>( a.Keys );

  Debug.Assert( 8 == keys.Count,
    "expected eight vertices for a rectangular column" );

  keys.Sort( Util.Compare );

  foreach( XYZ p in keys )
  {
    Debug.Assert( 3 == a[p],
      "expected every vertex of solid to appear in exactly three faces" );

    vertices.Append( p );
  }
}
```

The two assertions checking the total number of vertices, 8, and the number of coincident vertices in each corner, 3, are only valid for this special case, where I am looking at quadrilateral columns.
They will be removed in a future version, if I make use of this method for more general cases.

With this code in place, I can easily retrieve a sorted list of each of the solids' vertices for display and comparison purposes.

I used the code you provided in your original comment as a basis for a new command
named CmdNestedInstanceGeo, specifically tailored to explore this situation.
It first iterates over the solids contained in the top level family instance geometry and lists their vertices.
It then explores the family instance components and lists their names and location points.
Here is the code for the Execute methods that implements this:

```python
Application app = commandData.Application;
Document doc = app.ActiveDocument;

List<RvtElement> a = new List<RvtElement>();

if( !Util.GetSelectedElementsOrAll( a, doc,
  typeof( FamilyInstance ) ) )
{
  Selection sel = doc.Selection;
  message = ( 0 < sel.Elements.Size )
    ? "Please select some family instances."
    : "No family instances found.";
  return CmdResult.Failed;
}
FamilyInstance inst = a[0] as FamilyInstance;

Options opts = app.Create.NewGeometryOptions();
GeoElement geoElement = inst.get\_Geometry( opts );

GeometryObjectArray a1 = geoElement.Objects;
int n = a1.Size;

Debug.Print(
  "Family instance geometry has {0} geometry object{1}{2}",
  n, Util.PluralSuffix( n ), Util.DotOrColon( n ) );

int i = 0;
foreach( GeometryObject o1 in a1 )
{
  GeoInstance geoInstance = o1 as GeoInstance;
  if( null != geoInstance )
  {
    GeoElement symbolGeo = geoInstance.SymbolGeometry;
    GeometryObjectArray a2 = symbolGeo.Objects;
    foreach( GeometryObject o2 in a2 )
    {
      Solid s = o2 as Solid;
      if( null != s && 0 < s.Edges.Size )
      {
        XYZArray vertices = app.Create.NewXYZArray();
        GetVertices( vertices, s );
        n = vertices.Size;

        Debug.Print( "Solid {0} has {1} vertices{2} {3}",
          i++, n, Util.DotOrColon( n ),
          Util.PointArrayString( vertices ) );
      }
    }
  }
}
ElementSet components = inst.Symbol.Family.Components;
n = components.Size;

Debug.Print(
  "Family instance symbol family has {0} component{1}{2}",
  n, Util.PluralSuffix( n ), Util.DotOrColon( n ) );

foreach( RvtElement e in components )
{
  LocationPoint lp = e.Location as LocationPoint;
  Debug.Print( "{0} at {1}",
    Util.ElementDescription( e ),
    Util.PointString( lp.Point ) );
}
return CmdResult.Failed;
```

It accesses the one and only family instance in the file and stores it in 'inst'.
It then iterates over its geometry, extracts the vertices from each solid, and lists them in the debug output window.
Here is the result of that listing:

```
Family instance geometry has 1 geometry object:
Solid 0 has 8 vertices: (32.15,8.02,0), (32.15,8.02,13.12), (32.15,9.57,0), (32.15,9.57,13.12), (34.15,8.02,0), (34.15,8.02,13.12), (34.15,9.57,0), (34.15,9.57,13.12)
Solid 1 has 8 vertices: (28.24,3.23,0), (28.24,3.23,13.12), (28.24,4.79,0), (28.24,4.79,13.12), (30.24,3.23,0), (30.24,3.23,13.12), (30.24,4.79,0), (30.24,4.79,13.12)
Solid 2 has 8 vertices: (16,7.4,0), (16,7.4,13.12), (16,8.96,0), (16,8.96,13.12), (18,7.4,0), (18,7.4,13.12), (18,8.96,0), (18,8.96,13.12)
Solid 3 has 8 vertices: (12.08,2.61,0), (12.08,2.61,13.12), (12.08,4.17,0), (12.08,4.17,13.12), (14.09,2.61,0), (14.09,2.61,13.12), (14.09,4.17,0), (14.09,4.17,13.12)
Solid 4 has 8 vertices: (10.16,-4.19,0), (10.16,-4.19,13.12), (10.16,-2.19,0), (10.16,-2.19,13.12), (12.16,-4.19,0), (12.16,-4.19,13.12), (12.16,-2.19,0), (12.16,-2.19,13.12)
```

These lines are probably too long to be displayed in their entirety, but the main thing is to see that we have five identical solids which only differ from each other by a translation. You can see the full lines by copying and posting them into an editor.

The command next iterates over the family instance symbol family components, just like you suggested, and lists each one of these together with its location point:

```
Family instance symbol family has 3 components:
Columns M_Rectangular Column <2384 610 x 610mm> at (11.16,-3.19,0)
Furniture myFamily1 <3525 myFamily1> at (-2.48,2.65,0)
Furniture myFamily2 <4665 myFamily2> at (13.68,3.26,0)
```

As you can see, the column instance C0 and the two furniture family instances myFamily1 and myFamily2 are listed.
Diving into myFamily1 in the debugger and again traversing into its family instance components, I see the two nested columns C1 and C2, and so on.
We could enhance the code to traverse this structure recursively.

If you look at the location points of these instances and compare them with the offsets between the five solids, you can determine the correspondence between the internal nested structure and the detailed geometrical representation.

In this case, we defined our families extremely haphazardly, which makes it harder to understand the results. In a well designed family, I would expect both the structure and the locations of internal components to be less arbitrary.

I hope that this exploration helps you understand the structure, and provides some useful tools to explore it further.

Here is
[version 1.0.0.32](zip/bc10032.zip)
of the complete Visual Studio solution with the new command.