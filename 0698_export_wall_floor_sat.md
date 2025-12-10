---
post_number: "0698"
title: "Export Walls and Floors to SAT"
slug: "export_wall_floor_sat"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'revit-api', 'transactions', 'views', 'walls']
source_file: "0698_export_wall_floor_sat.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0698_export_wall_floor_sat.html"
---

### Export Walls and Floors to SAT

Happy New Year 2012 and welcome back to our explorations of the Revit API!

I had a nice winter break, and I hope you did too.

Let us start again by picking up an idea I recently mentioned for
[exporting a Revit element to an ACIS SAT file](http://thebuildingcoder.typepad.com/blog/2011/12/export-solid-to-acis.html).
Now Ishwar Nagwani presents a sample add-in implementing this idea to export all floor and wall elements to individual SAT files.
In his words:

Based on Miro's suggestion to create SAT file from individual elements, I wrote a sample to export all wall and floor elements to individual SAT files.
For each wall and floor element, all other elements are made temporarily invisible in the current view, and the current view with a single visible wall or floor element is exported to a SAT file.
The SAT files are saved in the system defined temp folder.

Note: Do not save the Revit file after export, since the document has been marked dirty due to committing transactions.

Here is Ishwar's code, with some slight editing by me.
It defines three methods:

- GetAllModelElements retrieves all model elements, which in this case we define as all non-ElementType elements having a valid category with valid material quantities assigned to it.- CreateSatFileFor, which temporarily hides all other elements and exports an individual floor or wall to SAT.- Execute, the external command mainline implementation.

Here is the code for GetAllModelElements to retrieve all visible elements that need to be hidden:
```csharp
IList<Element> GetAllModelElements( Document doc )
{
  List<Element> elements = new List<Element>();

  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType();

  foreach( Element e in collector )
  {
    if( null != e.Category
      && e.Category.HasMaterialQuantities )
    {
      elements.Add( e );
    }
  }
  return elements;
}
```

This is the main worker method CreateSatFileFor:
```csharp
void CreateSatFileFor(
  Element e,
  IList<Element> allElements,
  string filename\_prefix )
{
  Document doc = e.Document;

  // Keep this element visible and
  // hide all other model elements

  Transaction trans = new Transaction( doc );
  trans.Start( "Hide Elements" );

  // Create element set other than current wall/floor

  ElementSet eset = new ElementSet();

  foreach( Element ele in allElements )
  {
    if( e.Id.IntegerValue != ele.Id.IntegerValue )
    {
      if( ele.CanBeHidden( doc.ActiveView ) )
        eset.Insert( ele );
    }
  }

  // Hide all elements other than current
  // one in current view

  doc.ActiveView.Hide( eset );

  // Commit the transaction so that
  // visibility is affected

  trans.Commit();

  // Export the ActiveView containing current
  // element to SAT file

  SATExportOptions satExportOptions
    = new SATExportOptions();

  ViewSet vset = new ViewSet();
  vset.Insert( doc.ActiveView );

  // Get the material information

  IEnumerator<Material> mats =
    e.Materials.Cast<Material>().GetEnumerator();

  // Get the last material, as walls may
  // have multiple materials assigned

  Material mat = null;

  while( mats.MoveNext() )
  {
    mat = mats.Current;
  }

  // Get temp folder path for saving SAT files

  string folder = System.IO.Path.GetTempPath();

  string filename = filename\_prefix
    + "-" + e.Id.ToString()
    + "-" + mat.Name
    + "-" + mat.Id.ToString()
    + ".sat";

  doc.Export( folder, filename, vset,
    satExportOptions );

  // After export make all model elements visible

  trans = new Transaction( doc );
  trans.Start( "Unhide Elements" );
  doc.ActiveView.Unhide( eset );
  trans.Commit();
}
```

Finally, here is the Execute mainline collecting the walls and floors and driving the worked method:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  try
  {
    // Get all elements with material
    // for hiding all except current

    IList<Element> allElements
      = GetAllModelElements( doc );

    // Process all walls

    FilteredElementCollector col =
      new FilteredElementCollector( doc )
        .WhereElementIsNotElementType()
        .OfCategory( BuiltInCategory.OST\_Walls );

    foreach( Wall wall in col )
    {
      CreateSatFileFor( wall, allElements, "wall" );
    }

    // Process all floors

    col = new FilteredElementCollector( doc )
      .WhereElementIsNotElementType()
      .OfCategory( BuiltInCategory.OST\_Floors );

    foreach( Floor floor in col )
    {
      CreateSatFileFor( floor, allElements, "floor" );
    }
    return Result.Succeeded;
  }
  catch
  {
    TaskDialog.Show( "Error", "Could not create SAT Files" );
    return Result.Failed;
  }
}
```

Here is
[ExportToSat.zip](zip/ExportToSat.zip) containing
the complete source code, Visual Studio solution and add-in manifest of this external command.

Many thanks to Ishwar for putting together and sharing this!