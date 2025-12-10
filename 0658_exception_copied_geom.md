---
post_number: "0658"
title: "Exception on Copied Geometry"
slug: "exception_copied_geom"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'python', 'references', 'revit-api', 'transactions', 'views', 'walls']
source_file: "0658_exception_copied_geom.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0658_exception_copied_geom.html"
---

﻿

### Exception on Copied Geometry

I have heard about some issues with different types of geometry access throwing an exception.
As far as we know, there is a simple solution for most of them.
Here is an example:

**Question:** I have a piece of code iterating over Revit elements and extracting their geometry.
When run in release mode, it is causing an exception to be thrown.

I tested this in the sample model rac\_basic\_sample\_project.rvt, but it also throws a similar exception in all other files I tried.

Here is the code that is causing the problem:
```python
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
  protected bool ParseElements(
    FilteredElementCollector elements )
  {
    foreach( Element e in elements )
    {
      GeometryElement geoElement
        = e.get\_Geometry( new Options() );

      if( geoElement == null )
      {
        continue;
      }

      foreach( GeometryObject obj in geoElement.Objects )
      {
        if( obj is GeometryInstance )
        {
          GeometryInstance inst = obj
            as GeometryInstance;

          GeometryElement g2 = inst.SymbolGeometry;

          if( g2 != null )
          {
            Transform transformation = inst.Transform;

            g2 = g2.GetTransformed( transformation );

            foreach( GeometryObject obj2 in g2.Objects )
            {
              if( obj2 is Solid )
              {
                Solid solid = obj2 as Solid;

                foreach( Face face in solid.Faces )
                {
                  if( face == null ) continue;

                  Material material = e.Document
                    .Settings.Materials.get\_Item(
                      face.MaterialElementId );
                }
              }
            }
          }
          continue;
        }

        // obj is not GeometryInstance, check for solid

        Solid solid2 = obj as Solid;

        if( solid2 == null ) continue;

        foreach( Face face in solid2.Faces )
        {
          if( face == null ) continue;

          // This line which looks as if it should be read-only throws
          // "Attempt to modify the model outside of transaction.":

          Material material
            = e.Document.Settings.Materials.get\_Item(
              face.MaterialElementId );
        }
      }
    }
    return true;
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIDocument uidoc = commandData.Application.ActiveUIDocument;
    Document doc = uidoc.Document;

    View3D activeView = doc.ActiveView as View3D;

    if( activeView == null )
    {
      message = "Please run this in a 3D view";
      return Result.Failed;
    }

    FilteredElementCollector collector
      = new FilteredElementCollector(
        doc, activeView.Id );

    string s = "ParseElements Exception";

#if DEBUG
    string debug\_or\_release = "Debug";
#else
    string debug\_or\_release = "Release";
#endif

    // Transaction is required by Materials.get\_Item!?

    Transaction t = new Transaction( doc );
    t.Start( s );

    TaskDialog.Show( s, "Start ParseElements "
      + debug\_or\_release );

    ParseElements( collector );

    TaskDialog.Show( s, "End ParseElements "
      + debug\_or\_release );

    t.Commit();

    return Result.Succeeded;
  }
}
```

This code throws an exception saying "System.Runtime.InteropServices.SEHException (0x80004005): External component has thrown an exception".

If I comment out the statement 'g2 = g2.GetTransformed(transformation)', the exception is not thrown.

The exception is not also thrown in debug mode, only in release.

**Answer:** I tested your code in Revit Architecture 2012, Build 20110622\_0930, Update Release 1 and was unable to reproduce the issue, either in debug or release mode, either inside or outside the debugger.

After discussing the issue further with the development team, I received two pieces of good news for you:

Firstly, this issue can indeed happen with copied geometry.
By reassigning g2 to g2.Transformed(), you have released the handle for the original g2.
This may affect other handles you had previous obtained from g2, because all of the handles are tied together in native code.

The workaround is to make sure to keep the original g2 or some other GeometryElement handle in scope.
One way to achieve this is to assign it with a C# 'using' statement and not overwrite it.

Secondly, some issues related to the geometry handles have been fixed in Revit 2012 update release 1.
Internally, the Revit geometry handles have been revamped.
That explains why I was unable to reproduce the problem on my build.

That is a clear and succinct explanation, I think, which should enable you to avoid the problem completely.
It also assures you that you may revert to being a bit more carefree in future versions of Revit, and do not have to take such care retaining the handles you receive.

By the way, similar issues did come up from time to time in the past as well, and the resolution was always the same:

#### Exception Using Tessellate

**Question:** Calling the Tessellate method on edges sometimes causes the following error:

```
Exception: System.Runtime.InteropServices.SEHException
Message: External component has thrown an exception.
...
```

The error happens when retrieving a list of points from beam and joist elements.

It does not happen all the time.
I can run the add-in ten times on a specific project, and the exception may pop up just once or twice.

**Answer:** This is due to a similar issue with the lifetime of the GeometryInstance object as described above.
The scope workaround fixes it.

#### Exception Iterating through SolidArray

**Question:** I am creating an exporter which takes solid geometry from Revit and exports it to another application.
For each element selected, the tool finds its solid geometry and then parses the solid geometry's faces.
First I create an empty SolidArray and pass it by reference into a helper method taken from one of the Revit API samples.
Once the SolidArray is filled, I attempt to iterate over those solids, then over each solid's faces.
I get a SEHException at random iterations on the SolidArray.
Is there another way to recursively extract solid geometries from Family Instances without this error?

**Answer:** Looking at this in depth, we found that the crash is caused by the memory garbage collection.
The solid handle you retrieved was garbage collected before you actually made use of it.
This is a problem for solids which you copy from the original ones obtained from Revit.
The workaround is to ensure that the solids stay in scope by putting them in a suitable stable container.

#### Exception Iterating Over Elements

**Question:** I am developing an add-in for Revit that exports a file to another application.
In essence, it processes all geometry objects and places them into data structures that the other application can use.

The initial versions worked perfectly, but upon trying to make the add-in more efficient by adding a background worker thread to perform the bulk of the work and more user friendly by adding a progress bar I began encountering exceptions being generated from within Revit API methods.
I am assuming these exceptions were caused by the addition of the background worker thread and the progress bar.
The exceptions are only thrown when processing more complex geometry like families.
Simple elements like floors, walls, roofs, topological surfaces and site components all work fine.

In addition, if Revit is launched from the Visual Studio debugger, the add-in works fine.
If Revit is launched directly and loads the debug version of the add-in assembly, it also works.
The exceptions are thrown only when using the release version of the DLL and outside the debugger.

**Answer:** Probably the solid handles you retrieve are being garbage collected before you use them in your subroutine accessing the solid object.
As stated above, you simply need to ensure that the solid handles stay in scope.