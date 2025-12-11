---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.401895'
original_url: https://thebuildingcoder.typepad.com/blog/0124_cylindrical_column.html
post_number: '0124'
reading_time_minutes: 5
series: structural
slug: cylindrical_column
source_file: 0124_cylindrical_column.htm
tags:
- csharp
- elements
- family
- geometry
- parameters
- revit-api
- selection
- walls
- structural
title: Cylindrical Column
word_count: 1097
---

### Cylindrical Column

**Question:**
How can I detect whether a column is round in Revit Structure 2010?

I implemented a solution that worked in Revit Structure 2009, but it cannot be used in the 2010 SDK, because the SolidForms and CurveLoop methods no longer exist:

```csharp
bool IsColumnRound(
  FamilySymbol symbol )
{
  GenericFormSet solid = symbol.Family.SolidForms;
  GenericFormSetIterator i = solid.ForwardIterator();
  i.MoveNext();
  Extrusion extr = i.Current as Extrusion;
  CurveArray cr = extr.Sketch.CurveLoop;
  CurveArrayIterator i2 = cr.ForwardIterator();
  i2.MoveNext();
  String s = i2.Current.GetType().ToString();
  return s.Contains( "Arc" );
}
```

**Answer:**
I can think of several different approaches to analyse the cross section of a column family symbol:

1. The Family.SolidForms property.
2. The family instance solid geometry.
3. The analytical model swept profile.

The last of these, number 3, can be used in Revit Structure, but not in the other flavours of Revit.

The one you describe is the first approach, which can be used in Revit 2009, but not 2010.
It makes use of the Revit 2009 API Family.SolidForms property:

*SolidForms, a set of all solid forms that belong to a Family.
Returns a read only set of all the solid forms that belong to a particular family.*

In the Revit 2010 API help file, the section on the Family Creation API in the What's New chapter lists a host of new methods for working with families and states:

*These methods can be used to examine the contents of the family in terms of its elements, parameters and types;
as a result, the properties of Family which access the contents have been removed (Family.SolidForms, Family.VoidForms, Family.Components, Family.LoadedSymbols, Family.Others).*

We could probably rewrite your original code using the new Revit 2010 family API to analyse the shapes defined in the family symbol.

I find the approach number 2 the simplest, because it operates directly on the visible family instance geometry, and this approach can be reused in a variety of situations.
We have already discussed several such applications, in the posts on

- [Slab boundary](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html).
- [Slab side faces](http://thebuildingcoder.typepad.com/blog/2008/11/slab-side-faces.html).
- [Wall elevation profile](http://thebuildingcoder.typepad.com/blog/2008/11/wall-elevation-profile.html).
- [2D polygon areas and outer loop](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html).
- [3D polygon areas](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html).

Before doing anything further, it is always useful to analyse the structure and data of a Revit element using
[RvtMgdDbg](http://thebuildingcoder.typepad.com/blog/2009/02/rvtmgddbg.html).
It can be used to analyse the database contents and the element data, including its geometry.
I do not see any immediate access to the family symbol geometry through the API, but looking at a column instance, the geometry is right there.
For example, I loaded and inserted an instance of the round steel column family, 76.1x3.2CHS.
Selecting the instance in the user interface and using Add-Ins > RvtMgdDbg > Options > Snoop Current Selection > Geometry > Objects > Instance > Symbol Geometry > Objects > Faces brings me to a list of two planar and four cylindrical faces.
There are four cylindrical ones for two reasons: first, there column is hollow, so there is an inner and an outer face.
Secondly, the cross section is defined by two arcs instead of a circle, so each of the inner and outer faces is split into two.
Here is a screen snapshot of exploring a round structural column in Revit Architecture 2010:

![Round column faces](img/_round_column_faces.png)

I implemented a sample command CmdColumnRound which prompts you to select a column instance in the model and reports back to you whether it is cylindrical or not.
To decide whether this is the case, it uses a very simple approach that occurred to me during the implementation: in all cases that I know of, if the solid of a column includes one single cylindrical face, the entire column is cylindrical.
As long as this assumption holds, we can simplify the criteria for deciding whether the column is cylindrical to simply iterating over all its faces and deciding 'Yes' as soon as a cylindrical face is encountered.
Obviously, if you have more complex columns in your model which include a cylindrical face in a non-cylindrical column, this assumption no longer holds.

First, here is a helper predicate to determine whether a given Revit element 'e' looks like it might be a column family instance:

```csharp
bool IsColumn( RvtElement e )
{
  return e is FamilyInstance
    && null != e.Category
    && e.Category.Name.ToLower().Contains( "column" );
}
```

Here is the code for the Execute method:

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

Selection sel = doc.Selection;
RvtElement column = null;

if( 1 == sel.Elements.Size )
{
  foreach( RvtElement e in sel.Elements )
  {
    column = e;
  }
  if( !IsColumn( column ) )
  {
    column = null;
  }
}

if( null == column )
{
  sel.Elements.Clear();
  sel.StatusbarTip = "Please select a column";
  if( sel.PickOne() )
  {
    ElementSetIterator i
      = sel.Elements.ForwardIterator();
    i.MoveNext();
    column = i.Current as RvtElement;
  }
  if( !IsColumn( column ) )
  {
    message = "Please select a single column instance";
  }
}

if( null != column )
{
  Options opt = app.Create.NewGeometryOptions();
  GeoElement geo = column.get\_Geometry( opt );
  GeometryObjectArray objects = geo.Objects;
  GeoInstance i = null;
  foreach( GeometryObject obj in objects )
  {
    i = obj as GeoInstance;
    if( null != i )
    {
      break;
    }
  }
  if( null == i )
  {
    message = "Unable to obtain geometry instance";
  }
  else
  {
    bool isCylindrical = false;
    geo = i.SymbolGeometry;
    objects = geo.Objects;
    foreach( GeometryObject obj in objects )
    {
      Solid solid = obj as Solid;
      if( null != solid )
      {
        foreach( Face face in solid.Faces )
        {
          if( face is CylindricalFace )
          {
            isCylindrical = true;
            break;
          }
        }
      }
    }
    message = string.Format(
      "Selected column instance is{0} cylindrical",
      ( isCylindrical ? "" : " NOT" ) );
  }
}
return CmdResult.Failed;
```

The first part of the code demonstrates how to check whether a column has been preselected prior to running the command, and prompting the user to select one interactively otherwise.

Once the column instance has been determined, we obtain its geometry and extract the geometry instance from it. From that, we query the symbol geometry, search for its solid, query its faces, and decide that the column is round if a cylindrical face is encountered.

We make use of the message argument passed in to the Execute method to display the result of the analysis in the Revit user interface, and we return the command result Failed to ensure that the message is displayed and the Revit database is not marked as modified.

Here is
[version 1.0.0.29](http://thebuildingcoder.typepad.com/blog/files/bc10029.zip)
of the complete Visual Studio solution with the new CmdColumnRound command.