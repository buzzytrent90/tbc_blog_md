---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: code_example
optimization_date: '2025-12-11T11:44:13.262229'
original_url: https://thebuildingcoder.typepad.com/blog/0032_model_lines.html
post_number: '0032'
reading_time_minutes: 4
series: general
slug: model_lines
source_file: 0032_model_lines.htm
tags:
- elements
- geometry
- python
- revit-api
title: Model Line Creation
word_count: 700
---

### Model Line Creation

In the
[previous post](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html),
we determined the slab boundary polygons and displayed the resulting polygon data graphically for debugging and better understanding by creating model line segments for them in the Revit database. We did not go into any detail about the code to create those model lines, which is an interesting little topic all in itself. It is packaged in a utility class named Creator, since it encapsulates a small subset of the functionality provided by the Autodesk.Revit.Creation namespace. This post presents and discusses this model line creator. Here is the code for it:

```python
class Creator
{
  // these are
  // Autodesk.Revit.Creation
  // objects!
  Application \_app;
  Document \_doc;

  public Creator(
    Autodesk.Revit.Application app )
  {
    \_app = app.Create;
    \_doc = app.ActiveDocument.Create;
  }

  SketchPlane NewSketchPlanePassLine(
    Line line )
  {
    XYZ p = line.get\_EndPoint( 0 );
    XYZ q = line.get\_EndPoint( 1 );
    XYZ norm;
    if( p.X == q.X )
    {
      norm = XYZ.BasisX;
    }
    else if( p.Y == q.Y )
    {
      norm = XYZ.BasisY;
    }
    else
    {
      norm = XYZ.BasisZ;
    }
    Plane plane = \_app.NewPlane(
      norm, p );

    return \_doc.NewSketchPlane( plane );
  }

  void CreateModelLine( XYZ p, XYZ q )
  {
    if( p.AlmostEqual( q ) )
    {
      throw new ArgumentException(
        "Expected two different points." );
    }
    Line line = \_app.NewLine( p, q, true );
    if( null == line )
    {
      throw new Exception(
        "Geometry line creation failed." );
    }
    \_doc.NewModelCurve( line,
      NewSketchPlanePassLine( line ) );
  }

  public void DrawPolygons(
    List<List<XYZ>> loops )
  {
    XYZ p1 = XYZ.Zero;
    XYZ q = XYZ.Zero;
    bool first;
    foreach( List<XYZ> loop in loops )
    {
      first = true;
      foreach( XYZ p in loop )
      {
        if( first )
        {
          p1 = p;
          first = false;
        }
        else
        {
          CreateModelLine( p, q );
        }
        q = p;
      }
      CreateModelLine( q, p1 );
    }
  }
}
```

This code and the idea to create these kind of model lines for debugging purposes is actually demonstrated by the Revit SDK sample Openings, in its module BoundingBox.cs, which implements the two methods NewSketchPlanePassLine() and NewModelLine(), upon which the methods defined above are based.

Some things to note about this code:

The creation of new instances of many Revit API classes is not performed by the standard constructur, but by special Revit API classes and their methods in the Autodesk.Revit.Creation namespace. The two most important classes defined in the namespace are Application and Document. Note that these are completely different classes than the ones with the same names in the Autodesk.Revit namespace. The latter refer to the normal Revit application and document, whereas the former are used exclusively to instantiate new Revit API object instances. As the help file puts it, the Application creation object is used to create new instances of utility objects, and the Document creation object is used to create new instances of elements within the Autodesk Revit project.

The creation of a Revit database element is often a two-step process: first a geometry object is created to define its geometrical properties, and then a database object is created from that. The geometrical object is memory resident and independent of the database, so it is instantiated by the application creation object. The database object needs to be properly hooked up with the rest of the database, so obviously requires the document creation object. These two objects are stored in the private member variables \_app and \_doc, and set up by the constructor. Repeat: these are not instances of the Application and Document objects from the Autodesk.Revit namespace, but from the Autodesk.Revit.Creation one.

Revit elements are often linked to other database objects. In this case, the creation of a model line requires a sketch plane to reside on. The sketch plane must contain the line, otherwise unexpected behaviour might occur. To keep it simple, we create a separate sketch plane for every single model line, to ensure that this requirement is fulfilled, using NewSketchPlanePassLine().

Finally, we have the method DrawPolygons(), which is actually custom designed to handle the polygonal slab boundary loop data that we assembled in CmdSlabBoundary, but we have still kept it as generic as possible.

I find the creation of model lines for debugging purposes an efficient way to better understand what my code is actually doing. It is much more reliable to look at these graphical helper objects than try to read and analyse raw coordinate data.