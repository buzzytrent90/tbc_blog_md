---
post_number: "0298"
title: "Retrieving Column and Stair Geometry"
slug: "column_stair_geometry"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'revit-api', 'walls']
source_file: "0298_column_stair_geometry.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0298_column_stair_geometry.html"
---

### Retrieving Column and Stair Geometry

We have had several questions related to the retrieval of the geometry of stair elements in the past.
We already explored the geometry of various other elements, e.g.
[slab boundary](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html),
[slab side faces](http://thebuildingcoder.typepad.com/blog/2008/11/slab-side-faces.html),
[wall elevation profile](http://thebuildingcoder.typepad.com/blog/2008/11/wall-elevation-profile.html),
[2D polygon areas and outer loop](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html),
[3D polygon areas](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html),
[cylindrical columns](http://thebuildingcoder.typepad.com/blog/2009/04/cylindrical-column.html), and
[MEP ducts](http://thebuildingcoder.typepad.com/blog/2010/01/rectangular-duct-corners.html).

Here is now a solution by Joe Ye which may shed some new light on this issue.
The main point is that all elements which are represented by instances of a symbol may either have their own local geometry assigned to them, if it is unique in some way, or they may be reusing the standard geometry defined for the family type, transformed to their local insertion point, if it can be reused unmodified and thus shared with all other instances.

**Question:** I am trying to access the geometry data of different building elements in order to rebuilt them from their meshes and convert them to another format.
I had no problems converting walls, but some other objects don't provide access to their geometry data in the same way.

Problem #1: Columns.
I am filtering out all columns as FamilyInstances.
When I use the sample Revit 'Urban House' model, I can access their geometry, get the related solid and access the mesh from its faces.

On the other hand, when I apply the same approach to another model of my own, this is not possible.
In this model, the attempt to cast the GeometryObjects to solids returns null.
Is there maybe something wrong with this model or the families used in it?

Another Problem here is filtering for columns which are not of the structural type StructuralType.Column.
I am doing this by accessing the Family.Instance.Category.Name which is not very elegant.

Problem #2: Stairs.
The problem I have with stairs is that I can cast the GeometryObjects to solids, but they do not contain any faces.
Is there maybe a different way to access the geometry of stairs, so that I can create a mesh from it?

**Answer:** The issue with the columns is that the ones used in your own sample model file are standalone columns which do not join or intersect with any other elements.
If a column or some other FamilyInstance element is joined to another element, its geometry might be changed by this operation.
Therefore, each instance needs to maintain its own specific copy of the geometry.

On the other hand, a standalone column or FamilyInstance object that does not cut any other element can reuse the unmodified geometry defined by its family symbol.
This helps reduce file size, since it avoids creating and maintaining multiple copies of the same geometry.

We therefore need to check whether the family instance geometry contains any solid of its own.
If not, then it may be a standalone element, and we need to look at the symbol's geometry instead.
The standalone element type geometry is accessible through Geometry > Instance > SymbolGeometry.
The geometry retrieved then needs to be transformed from the symbol definition coordinate system to the instance coordinate system by applying the transform retrieved from Geometry > Instance > Transform.

Here is a pseudo code example demonstrating this:
```csharp
void generateSingleElement( RvtElement e )
{
  GeoElement geo = e.get\_Geometry( geomOption );

  if( geo != null )
  {
    GeometryObjectArray arr = geo.Objects;

    foreach( GeometryObject obj in arr )
    {
      Solid geomSolid = obj as Solid;

      if( null != geomSolid ) //+
      {
        // get geometry from this solid
        // ...
      }
      else if( obj is GeoInstance )
      {
        GeoInstance geoInst = obj as GeoInstance;

        GeoElement geoElem = geoInst.SymbolGeometry;

        Transform transform = geoInst.Transform;

        GeometryObjectArray arr2 = geoElem.Objects;

        // use the same method as above to obtain
        // the nested geometry. please don't
        // forget to convert with the transform.
        // ...
      }
    }
  }
}
```

The other problem you mention is filtering the columns which do not have the structural type StructuralType.Column.
You need to be aware of the fact that a StructuralType is only applicable to structural elements, for example structural columns and structural beams.
Normal architectural column instances will not have a structural type assigned to them, so you cannot use this method to filter for them.
In that case, you should use the more general approach of filtering for the desired combination of FamilyInstance type and column category.

Regarding your issue with stairs, the solution is the same as for the columns.

Many thanks to Joe for handling this solution!