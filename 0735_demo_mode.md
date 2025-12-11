---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.483640'
original_url: https://thebuildingcoder.typepad.com/blog/0735_demo_mode.html
post_number: '0735'
reading_time_minutes: 3
series: general
slug: demo_mode
source_file: 0735_demo_mode.htm
tags:
- csharp
- elements
- filtering
- references
- revit-api
- transactions
- views
title: Determine Revit Demo Mode
word_count: 574
---

### Determine Revit Demo Mode

Here is yet another interesting example of an apparent gap in the Revit API that can be easily filled using a little workaround.

We have seen numerous examples of performing a certain operation within a temporary transaction that is then rolled back to cancel it, such as to determine gross
[material quantities](http://thebuildingcoder.typepad.com/blog/2010/02/material-quantity-extraction.html) for
an element with openings, a
[host reference](http://thebuildingcoder.typepad.com/blog/2009/06/host-reference.html) and,
more generally, all
[object relationships](http://thebuildingcoder.typepad.com/blog/2010/03/object-relationships.html).

Here is an example of using a related but different approach: the trick consists in attempting to perform a certain operation and gracefully handling a specific expected failure condition, in ths case a specific exception being thrown, to determine whether Revit is running in demo mode:

**Question:** Is it possible to determine whether Revit runs in demo mode?
I need this for two different reasons:

- It is a licensing issue, since read-only viewing should not occupy a license.- Some add-in commands should be greyed out or hidden in the ribbon bar in demo mode.

**Answer:** You could try to save the model (after making a modification!) and then see if you get an InvalidLicenseException.

I implemented a little sample application TestDemoMode to test a simplified version of this idea.

Here is the test method I created:
```csharp
  static bool IsDemoMode( Document doc )
  {
    Application app = doc.Application;

    try
    {
      Transaction tx = new Transaction( doc );

      tx.Start( "Modify Document" );

      SketchPlane sp
        = new FilteredElementCollector( doc )
          .OfClass( typeof( SketchPlane ) )
          .FirstElement() as SketchPlane;

      Line line = app.Create.NewLineBound(
        XYZ.Zero, XYZ.BasisX );

      ModelCurve mc = doc.Create.NewModelCurve(
        line, sp );

      tx.Commit();

      string filename = Path.GetTempFileName();
      File.Delete( filename );

      doc.SaveAs( filename );
      File.Delete( filename );

      tx.Start( "Unmodify Document" );
      doc.Delete( mc );
      tx.Commit();

      return false;
    }
    catch( InvalidLicenseException ex )
    {
      return true;
    }
  }
```

This is the external command Execute mainline code:
```csharp
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    bool rc = IsDemoMode( doc );

    string s = string.Format(
      "This is {0}a Revit demo version.",
      rc ? "" : "not " );

    TaskDialog.Show( "Test Demo Mode", s );

    return Result.Succeeded;
  }
```

I have not tested it for both cases, though, since I do not have a demo version handy.

I did run it in a non-demo version, which was correctly reported:
![Not a demo version](img/not_demo_version.png)

I have also not added any checks to determine whether the write and save attempt may be failing due to the document being read-only, disk full, or anything like that.

A much better approach would be to create a new document from scratch and attempt to save that.

Anyway, here is
[TestDemoMode.zip](zip/TestDemoMode.zip) including
the source code, add-in manifest and full Visual Studio solution of the simplified test command.

A full-blown implementation requires a number of additional considerations and lots of testing, of course.
As said, the '1 new document' - '2 edit document' - '3 save as' strategy might be best.
Hopefully all exceptions thrown during the first two steps will have to do with the licensing.
Step three is less certain, since for instance the hard drive may be full.

Saving the current document is a really bad idea and might take a long time, without even thinking about detaching from central and keeping work-shares.