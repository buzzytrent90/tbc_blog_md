---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: code_example
optimization_date: '2025-12-11T11:44:15.151028'
original_url: https://thebuildingcoder.typepad.com/blog/1026_save_solid_to_file.html
post_number: '1026'
reading_time_minutes: 4
series: general
slug: save_solid_to_file
source_file: 1026_save_solid_to_file.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- python
- revit-api
- selection
- transactions
- views
title: Saving a Solid to a SAT File Implementation
word_count: 792
---

### Saving a Solid to a SAT File Implementation

During my recent vacation, I published a description by my colleague
[Akira Kudo](http://adndevblog.typepad.com/technology_perspective/akira-kudo.html) describing
how to research the Revit SDK samples to find all the required bits and pieces to solve the task of
[saving a temporary in-memory Revit solid to an external SAT file](http://thebuildingcoder.typepad.com/blog/2013/09/how-to-save-a-solid-to-a-file.html).

Victor Chekalin, or Виктор Чекалин, reacted to that and provides a complete Visual Studio project to demonstrate the full implementation of that functionality.

You can download it as a
[zip archive](https://github.com/vchekalin/SaveSolidToFileSample/archive/master.zip) or
clone the project from
[GitHub](https://github.com/vchekalin/SaveSolidToFileSample).

I grabbed Victor's sample, added a couple of trivial enhancements and included it as a new command CmdExportSolidToSat to The Building Coder sample collection.

It demonstrates the following steps:

- Retrieve all floors from the model.
- Retrieve the floor solids.
- Calculate the intersection solid.
- [Search for the metric mass family template file](#2).
- Create a new temporary family.
- Create a free form element from the intersection solid.
- Create a 3D view.
- Export to SAT.

#### Determining the Full Path of a Family Template

One of the steps implements functionality to find the full file path of a given family template file using a recursive directory search method:

```csharp
  /// <summary>
  /// Return the full path of the first file
  /// found matching the given filename pattern
  /// in a recursive search through all
  /// subdirectories of the given starting folder.
  /// </summary>
  string DirSearch(
    string start\_dir,
    string filename\_pattern )
  {
    foreach( string d in Directory.GetDirectories(
      start\_dir ) )
    {
      foreach( string f in Directory.GetFiles(
        d, filename\_pattern ) )
      {
        return f;
      }

      string f2 = DirSearch( d, filename\_pattern );

      if( null != f2 )
      {
        return f2;
      }
    }
    return null;
  }
```

With this method in place, I can determine the full path of the standard metric mass family template using a single statement like this:

```csharp
  // Search for the metric mass family template file

  string template\_path = DirSearch(
    app.FamilyTemplatePath,
    "Metric Mass.rft" );
```

#### CmdExportSolidToSat External Command Implementation

The other steps listed above are all pretty standard.

Here is the complete code of the external command:

```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;
  Selection sel = uidoc.Selection;

  // Retrieve all floors from the model

  var floors
    = new FilteredElementCollector( doc )
      .OfClass( typeof( Floor ) )
      .ToElements()
      .Cast<Floor>()
      .ToList();

  if( 2 != floors.Count )
  {
    message = "Please create two intersected floors";
    return Result.Failed;
  }

  // Retrieve the floor solids

  Options opt = new Options();

  var geometry1 = floors[0].get\_Geometry( opt );
  var geometry2 = floors[1].get\_Geometry( opt );

  var solid1 = geometry1.FirstOrDefault() as Solid;
  var solid2 = geometry2.FirstOrDefault() as Solid;

  // Calculate the intersection solid

  var intersectedSolid = BooleanOperationsUtils
    .ExecuteBooleanOperation( solid1, solid2,
      BooleanOperationsType.Intersect );

  // Search for the metric mass family template file

  string template\_path = DirSearch(
    app.FamilyTemplatePath,
    "Metric Mass.rft" );

  // Create a new temporary family

  var family\_doc = app.NewFamilyDocument(
    template\_path );

  // Create a free form element
  // from the intersection solid

  using( var t = new Transaction( family\_doc ) )
  {
    t.Start( "Add Free Form Element" );

    var freeFormElement = FreeFormElement.Create(
      family\_doc, intersectedSolid );

    t.Commit();
  }

  string dir = Path.GetTempPath();

  string filepath = Path.Combine( dir,
    "floor\_intersection\_family.rfa" );

  SaveAsOptions sao = new SaveAsOptions()
  {
    OverwriteExistingFile = true
  };

  family\_doc.SaveAs( filepath, sao );

  // Create 3D View

  var viewFamilyType
    = new FilteredElementCollector( family\_doc )
    .OfClass( typeof( ViewFamilyType ) )
    .OfType<ViewFamilyType>()
    .FirstOrDefault( x =>
      x.ViewFamily == ViewFamily.ThreeDimensional );

  View3D threeDView;

  using( var t = new Transaction( family\_doc ) )
  {
    t.Start( "Create 3D View" );

    threeDView = View3D.CreateIsometric(
      family\_doc, viewFamilyType.Id );

    t.Commit();
  }

  // Export to SAT

  var viewSet = new List<ElementId>()
  {
    threeDView.Id
  };

  SATExportOptions exportOptions
    = new SATExportOptions();

  var res = family\_doc.Export( dir,
    "SolidFile.sat", viewSet, exportOptions );

  return Result.Succeeded;
}
```

Here is the SAT file
[SolidFile.sat](zip/SolidFile.sat) generated
by the two intersecting floors example that I used to test the
[RvtClipper Boolean operations for 2D polygons](http://thebuildingcoder.typepad.com/blog/2013/09/boolean-operations-for-2d-polygons.html):

![SAT file reimported into Revit](img/import_sat.png)

It generates the following warning when reimported into Revit, but the missing pieces do not make any visibly detectable difference:

![SAT file import warning](img/import_sat_warning.png)

They do however affect what Revit considers the size of imported object, because if I zoom the model containing nothing but this SAT import to its extents, the actual geometry appears pretty miniscular:

![SAT file import extents](img/import_sat_extents.png)

That need not worry us here, though.

Here is
[version 2014.0.104.0](zip/bc_14_104_0.zip) of
The Building Coder samples source code, Visual Studio solution and RvtSamples include file including the new CmdExportSolidToSat command.

Very many thanks to Victor for providing this nice implementation!