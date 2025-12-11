---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.1
content_type: code_example
optimization_date: '2025-12-11T11:44:15.416094'
original_url: https://thebuildingcoder.typepad.com/blog/1154_directshape_min_size.html
post_number: '1154'
reading_time_minutes: 5
series: general
slug: directshape_min_size
source_file: 1154_directshape_min_size.htm
tags:
- elements
- family
- filtering
- geometry
- python
- revit-api
- rooms
- selection
- transactions
- windows
title: DirectShape Performance and Minimum Size
word_count: 1031
---

### DirectShape Performance and Minimum Size

One of the exciting API enhancements in Revit 2015 is the Import API functionality providing the new
[DirectShape and TessellatedShapeBuilder](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#3.09) classes.

These enable significant [performance enhancements](#2) and powerful new possibilities for generating libraries and importing geometry, especially repetitive shapes.

On the other hand, the same limitations on [minimum size](#3) apply as in the past.

#### DirectShape Performance

Regarding the performance, here is a first enthusiastic reaction and quick report from a developer after the initial introduction during the beta phase last December:

I examined the DirectShape functionality a little.

My results are very interesting.

I implemented a benchmark generating 'room solids'.

In the past, I had to create a new family for this, define its geometry using freeform elements, save the family, load it into the project and place an instance.

That method took about 2 hours for 1390 rooms and increased the file size by about 60 Kb per room.

My first alternative test implementation using DirectShape elements takes 40 seconds in total and increases the file size by 2 Kb per room.

This offers fantastic new possibilities to create quick "permanent" geometry!

Not to mention how much simpler the code is now!

Knowing the Revit API, there is one obvious limitation, though...

#### DirectShape Minimum Size

Here is a question that came up a couple of times now:

**Question:** I created a DirectShape in Revit 2015 from a collection of faces using TessellatedShapeBuilder.

The minimum length for edges accepted by Revit is 0.0026 feet. If I use 0.0025 feet Revit crashes.

Can I change this tolerance?

Or is any other way to construct DirectShape elements that accepts edges smaller than 0.0026 feet?

**Answer:** Unfortunately, the answer is no and no.

This tolerance cannot be changed, and I am not aware of any other way to create DirectShape elements to circumvent that limitation.

You
[have to think big](http://thebuildingcoder.typepad.com/blog/2009/07/think-big-in-revit.html) when
creating objects in Revit, to the chagrin of people wishing to represent very detailed metal and sub-millimetre wood construction.

The limitation in creating individual model line segments lies around 1/16 of an inch, and the same applies to DirectShape element edges.

I implemented a little external command to test this.

It defines a static Boolean switch variable named `Iterate_until_crash`.

It is set to false by default, in which case the external command simply creates a DirectShape tetrahedron with an edge length of 0.5 feet like this:

![DirectShape tetrahedron](img/direct_shape_min_size_1.png)

It reports its accomplished task like this in the Visual Studio debugging output window:

```
Creating DirectShape tetrahedron with side length 0.5...
```

When Iterate\_until\_crash is set to true, the code enters an infinite loop creating successive DirectShape elements in each iteration, each with half of the preceding edge length.

It throws and exception quite soon, though, so none of the shapes end up being committed, and the debug output now reports:

```
Creating DirectShape tetrahedron with side length 0.5...
Creating DirectShape tetrahedron with side length 0.25...
Creating DirectShape tetrahedron with side length 0.125...
Creating DirectShape tetrahedron with side length 0.0625...
Creating DirectShape tetrahedron with side length 0.03125...
Creating DirectShape tetrahedron with side length 0.015625...
Creating DirectShape tetrahedron with side length 0.0078125...
Creating DirectShape tetrahedron with side length 0.00390625...
Creating DirectShape tetrahedron with side length 0.001953125...
```

- A first chance exception of type 'Autodesk.Revit.Exceptions.InvalidOperationException' occurred in RevitAPI.dll
- Creating DirectShape tetrahedron with side length 0.001953125 threw exception 'could not create consistent vertex list'

The exception is passed back to the standard Revit failure handler:

![DirectShape minimum size](img/direct_shape_min_size_2.png)

Here is the entire external command implementation:

```python
#region Namespaces
using System;
using System.Linq;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Autodesk.Revit.UI.Selection;
#endregion

namespace DirectShapeMinSize
{
  [Transaction( TransactionMode.Manual )]
  public class Command : IExternalCommand
  {
    /// <summary>
    /// Set this to true to iterate through smaller
    /// and smaller tetrahedron sizes until we hit
    /// Revit's precision limit.
    /// </summary>
    static bool Iterate\_until\_crash = false;

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIApplication uiapp = commandData.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Application app = uiapp.Application;
      Document doc = uidoc.Document;

      // Find GraphicsStyle

      FilteredElementCollector collector
        = new FilteredElementCollector( doc )
          .OfClass( typeof( GraphicsStyle ) );

      GraphicsStyle style = collector.Cast<GraphicsStyle>()
        .FirstOrDefault<GraphicsStyle>( gs => gs.Name.Equals( "<Sketch>" ) );

      ElementId graphicsStyleId = null;

      if( style != null )
      {
        graphicsStyleId = style.Id;
      }

      // Modify document within a transaction

      using( Transaction tx = new Transaction( doc ) )
      {
        tx.Start( "Create DirectShape" );

        double length = 1; // foot

        try
        {
          do
          {
            length = 0.5 \* length;

            Debug.Print(
                "Creating DirectShape tetrahedron with side length {0}...",
                length );

            List<XYZ> args = new List<XYZ>( 3 );

            TessellatedShapeBuilder builder = new TessellatedShapeBuilder();

            builder.OpenConnectedFaceSet( false );

            args.Add( XYZ.Zero );
            args.Add( length \* XYZ.BasisX );
            args.Add( length \* XYZ.BasisY );
            builder.AddFace( new TessellatedFace( args, ElementId.InvalidElementId ) );

            args.Clear();
            args.Add( XYZ.Zero );
            args.Add( length \* XYZ.BasisX );
            args.Add( length \* XYZ.BasisZ );
            builder.AddFace( new TessellatedFace( args, ElementId.InvalidElementId ) );

            args.Clear();
            args.Add( XYZ.Zero );
            args.Add( length \* XYZ.BasisY );
            args.Add( length \* XYZ.BasisZ );
            builder.AddFace( new TessellatedFace( args, ElementId.InvalidElementId ) );

            args.Clear();
            args.Add( length \* XYZ.BasisX );
            args.Add( length \* XYZ.BasisY );
            args.Add( length \* XYZ.BasisZ );
            builder.AddFace( new TessellatedFace( args, ElementId.InvalidElementId ) );

            builder.CloseConnectedFaceSet();

            TessellatedShapeBuilderResult result
              = builder.Build(
                TessellatedShapeBuilderTarget.Solid,
                TessellatedShapeBuilderFallback.Abort,
                graphicsStyleId );

            // Pre-release code from DevDays

            //DirectShape ds = DirectShape.CreateElement(
            //  doc, result.GetGeometricalObjects(), "A", "B");

            //ds.SetCategoryId(new ElementId(
            //  BuiltInCategory.OST\_GenericModel));

            // Code updated for Revit UR1

            ElementId categoryId = new ElementId(
              BuiltInCategory.OST\_GenericModel );

            DirectShape ds = DirectShape.CreateElement(
              doc, categoryId, "A", "B" );

            ds.SetShape( result.GetGeometricalObjects() );

            ds.Name = "Test";
          }
          while( Iterate\_until\_crash && 0 < length );
        }
        catch( Exception e )
        {
          Debug.Print(
            "Creating DirectShape tetrahedron with side length {0} "
            + "threw exception '{1}'",
            length, e.Message );

          message = e.Message;
          return Result.Failed;
        }
        tx.Commit();
      }
      return Result.Succeeded;
    }
  }
}
```

For your convenience, here is
[DirectShapeMinSize.zip](zip/DirectShapeMinSize.zip) providing
the complete source code, Visual Studio solution and add-in manifest for the DirectShapeMinSize external command.