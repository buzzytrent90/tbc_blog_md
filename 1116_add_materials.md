---
post_number: "1116"
title: "Adding New Materials from List Updated"
slug: "add_materials"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'revit-api', 'sheets', 'transactions', 'windows']
source_file: "1116_add_materials.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1116_add_materials.html"
---

### Adding New Materials from List Updated

Back in the year 2010, I discussed a simple little system based on a Revit add-in named AddMaterials to
[generate new materials in a Revit project](http://thebuildingcoder.typepad.com/blog/2010/08/add-new-materials-from-list.html#2) based
on a list of required material properties stored in an Excel spreadsheet.

The input data includes:

- Material Name

- Code
- Title
- Strength

- Graphics

- RGB colour values
- Transparency
- Surface pattern
- Cut pattern

- Identity
  - Filter criteria
  - Descriptive information
  - Custom parameters

All the further workflow details are described in the
[original post](http://thebuildingcoder.typepad.com/blog/2010/08/add-new-materials-from-list.html#2),
of course.

The add-in was implemented for Revit 2011, and quite a bit of the required Revit API functionality has changed since then, so an update of this system seems in order.

Here are some of the things that have changed:

1. The document Settings class no longer provides the Materials and FillPatterns properties.
2. The Color class constructor requires RGB arguments.
3. The Material class SurfacePattern and CutPattern properties have been replaced by properties taking element ids.

These changes, that accumulated over three major releases of the Revit API, can be rather baffling for an API newbie.

Here are the methods I used to update to the Revit 2014 API functionality:

1. Use filtered element collectors to access the document materials and fill patterns, and LINQ to generate dictionaries mapping the element names to instances.
2. Provide RGB arguments to the Color constructor.
3. Set the material surface and cut patterns using element ids.

At the same time, I took this opportunity to clean up the code a bit, e.g. by eliminating global variables and removing unnecessary casts.

Here is the complete updated implementation of the AddMaterials add-in for Revit 2014:

```csharp
#region Namespaces
using System;
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
using Excel = Microsoft.Office.Interop.Excel;
#endregion

namespace AddMaterials
{
  [Transaction( TransactionMode.Manual )]
  public class Command : IExternalCommand
  {
    static string PluralSuffix( int i )
    {
      return 1 == i ? "" : "s";
    }

    public Result Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      UIApplication uiapp = commandData.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Document doc = uidoc.Document;

      // Create dictionary of existing
      // materials keyed by their name.

      Dictionary<string, Material> materials
        = new FilteredElementCollector( doc )
          .OfClass( typeof( Material ) )
          .Cast<Material>()
          .ToDictionary<Material, string>(
            e => e.Name );

      // Ditto for fill patterns.

      Dictionary<string, FillPatternElement> fillPatterns
        = new FilteredElementCollector( doc )
          .OfClass( typeof( FillPatternElement ) )
          .Cast<FillPatternElement>()
          .ToDictionary<FillPatternElement, string>(
            e => e.Name );

      try
      {
        Excel.Application excel
          = new Excel.Application();

        excel.Visible = false;

        string filename = "C:/RevitAPI/MaterialList.xlsx";

        Excel.Workbook workbook = excel.Workbooks.Open(
          filename, 0, true, 5, "", "", true,
          Excel.XlPlatform.xlWindows, "\t", false,
          false, 0, true, 1, 0 );

        Excel.Worksheet worksheet = (Excel.Worksheet)
          workbook.Worksheets.get\_Item( 1 );

        Excel.Range range = worksheet.UsedRange;

        int nRows = 0;
        int nMaterialAdded = 0;

        int iRow = 5;

        using( Transaction tx = new Transaction( doc ) )
        {
          tx.Start( "Add Materials" );

          while( null != range.Cells[iRow, 1].Value2 )
          {
            string matName = (string) range.Cells[iRow, 1].Value2;
            matName += " " + (string) range.Cells[iRow, 2].Value2;
            matName += " " + (string) range.Cells[iRow, 3].Value2;

            if( matName != null )
            {
              double red = (double) range.Cells[iRow, 4].Value2;
              double green = (double) range.Cells[iRow, 5].Value2;
              double blue = (double) range.Cells[iRow, 6].Value2;
              double transparency = (double) range.Cells[iRow, 8].Value2;
              string surPattern = (string) range.Cells[iRow, 9].Value2;
              string cutPattern = (string) range.Cells[iRow, 10].Value2;

              // Identity data of material class to duplicate

              string CSI = (string) range.Cells[iRow, 11].Value2;

              if( materials.ContainsKey( CSI ) )
              {
                Material materialCSI = materials[CSI];

                Material myMaterial
                  = materialCSI.Duplicate( matName );

                Color matColor = new Color(
                  Byte.Parse( red.ToString() ),
                  Byte.Parse( green.ToString() ),
                  Byte.Parse( blue.ToString() ) );

                myMaterial.Color = matColor;

                myMaterial.Transparency
                  = (int) transparency;

                myMaterial.SurfacePatternId
                  = fillPatterns[surPattern].Id;

                myMaterial.CutPatternId
                  = fillPatterns[cutPattern].Id;

                ++nMaterialAdded;
              }
            }
            ++nRows;
            ++iRow;
          }
          tx.Commit();
        }

        workbook.Close( true, null, null );
        excel.Quit();

        TaskDialog.Show(
          "Revit AddMaterials",
          string.Format(
            "{0} row{1} successfully parsed and "
            + "{0} material{1} added.",
            nRows, PluralSuffix( nRows ),
            nMaterialAdded,
            PluralSuffix( nMaterialAdded ) ) );

        return Result.Succeeded;
      }
      catch( Exception ex )
      {
        message = "Revit AddMaterials Exception:\n"
          + ex.ToString();

        return Result.Failed;
      }
    }
  }
}
```

Note the nice use of the generic ToDictionary method to convert from a filtered element collector to a dictionary in one fell swoop.

For the complete source code, Visual Studio solution and add-in manifest, please refer to the
[AddMaterials GitHub repository](https://github.com/jeremytammik/AddMaterials).

The version discussed above is
[release 2014.0.0.0](https://github.com/jeremytammik/AddMaterials/releases/tag/2014.0.0.0).

I hope you find this useful and instructive.

**Addendum:** This utility has been updated.
Please check out the
[enhancements in release 2014.0.0.1](http://thebuildingcoder.typepad.com/blog/2014/03/adding-new-materials-from-list-updated-again.html).