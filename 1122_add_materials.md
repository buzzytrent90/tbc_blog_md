---
post_number: "1122"
title: "Adding New Materials from List Enhancements"
slug: "add_materials"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'python', 'revit-api', 'sheets', 'transactions', 'views', 'windows']
source_file: "1122_add_materials.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1122_add_materials.html"
---

### Adding New Materials from List Enhancements

A weekend post, for a change.

There is just too much going on!

I recently
[reimplemented the AddMaterials add-in](http://thebuildingcoder.typepad.com/blog/2014/03/adding-new-materials-from-list-updated.html) for
Revit 2014, updating the simple little Revit 2011 system to
[generate new materials in a Revit project](http://thebuildingcoder.typepad.com/blog/2010/08/add-new-materials-from-list.html#2) based
on a list of required material properties stored in an Excel spreadsheet.

Luke submitted a
[comment](http://thebuildingcoder.typepad.com/blog/2014/03/adding-new-materials-from-list-updated.html?cid=6a00e553e16897883301a73d9bb66c970d#comment-6a00e553e16897883301a73d9bb66c970d) on
that, asking:

**Question:** I have built the solution and created an XLSX file.
The add-in runs and says that it has added materials, but I don't see them in the Material Browser in 2014.
Can you please provide an example of the Excel file so that I can ensure the formatting I'm using is correct?

**Answer:** Just as I suspected, taking a quick look at what is happening in the debugger immediately revealed the problem.

Two problems, in fact:

1. Your spreadsheet does indeed specify the required CSI values.
   However, these values need to refer to existing materials in your project file.
   If no such materials exist, the addition of the new materials will fail, because the system will not have a source material to copy.
2. The final report has a bug.
   It refers twice to the variable nRows, counting the number of spreadsheet rows read, and never to nMaterialAdded, the number of materials added. Therefore, your flawed input was not reported.

I started off by fixing the error message.

The original format string includes the placeholders {0} and {1} twice, and {2} and {3} not at all.

I fixed the final report formatting string like this:

```csharp
  string msg = string.Format(
    "{0} row{1} successfully parsed and "
    + "{2} material{3} added.",
    nRows, PluralSuffix( nRows ),
    nMaterialAdded, PluralSuffix(
      nMaterialAdded ) );

  TaskDialog.Show( "Revit AddMaterials", msg );
```

Now, when I run the add-in with your sample material spreadsheet in a default empty architectural project, I receive the following final report message:

![Zero materials added](img/add_materials_report_zero.png)

Next, I manually added the first one of the source materials requested by your input spreadsheet via Manage > Materials > Brick, Common > Duplicate and named it "CSI 03", as specified in the CSI column of your input spreadsheet.

Now, at least the ContainsKey method returns true for that material, and the duplication code is executed.

Unfortunately, that leads to the next error, saying that a dictionary key is not found:

![Dictionary key not found error](img/add_materials_key_not_found.png)

This is caused by the cutPattern variable, whose value is "N/A" and yet is used as a key into the materials dictionary.
That call obviously fails, because no material named "N/A" is defined.

I adapted the code to check the surPattern and cutPattern variables for "N/A" values before trying to retrieve materials for them, like this:

```csharp
  const string \_not\_available = "N/A";
  if( 0 < surPattern.Length
    && !surPattern.Equals( \_not\_available ) )
  {
    myMaterial.SurfacePatternId
      = fillPatterns[surPattern].Id;
  }

  if( 0 < cutPattern.Length
    && !cutPattern.Equals( \_not\_available ) )
  {
    myMaterial.CutPatternId
      = fillPatterns[cutPattern].Id;
  }
```

Now all problems in the code have been resolved and the "N/A" string is gracefully handled, I get a sensible reporting message stating that one material was added (provided one source material exists):

![One material added](img/add_materials_report_one.png)

There are still a couple of issues that make this system less than fool-proof, i.e., require a minimum intelligence on the side of the user:

- If the number of materials added is less than the number of spreadsheet rows read, some materials were not added, for some reason.
- If some of the requested materials were not added, it is left up to you to figure out which they were.

Experience shows that requiring awareness on the part of the user is a rather steep request :-)

The add-in can obviously easily be improved to fix these issues.

That should really be left as an exercise to you, dear reader.

I am sure I can trust you to take care of it.

I still decided to help a little bit more, by maintaining a list of the names of the materials added, instead of just an integer variable counting them:

```csharp
  List<string> materials\_added = new List<string>();
```

Then I can enhance the report like this:

```csharp
  int n = materials\_added.Count;

  string msg = string.Format(
    "{0} row{1} successfully parsed and "
    + "{2} material{3} added:",
    nRows, PluralSuffix( nRows ),
    n, PluralSuffix( n ) );

  TaskDialog dlg = new TaskDialog(
    "Revit AddMaterials" );

  dlg.MainInstruction = msg;

  dlg.MainContent = string.Join( ", ",
    materials\_added ) + ".";

  dlg.Show();
```

With that enhancement, the report now looks like this:

![Names listed of all materials added](img/add_materials_report_names.png)

It still remains the responsibility of the user to notice that more rows were specified and parsed than materials added, and to figure out which failed.
At least the names of the materials added successfully are listed.

For the sake of completeness, here is the entire source code of the updated external command:

```python
const string \_not\_available = "N/A";

const string \_input\_file\_name = "C:/RevitAPI/MaterialList.xlsx";

//const string \_input\_file\_name = "Z:/a/doc/revit/blog/zip/MaterialList.xlsx";

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

    Excel.Workbook workbook = excel.Workbooks.Open(
      \_input\_file\_name, 0, true, 5, "", "", true,
      Excel.XlPlatform.xlWindows, "\t", false,
      false, 0, true, 1, 0 );

    Excel.Worksheet worksheet = (Excel.Worksheet)
      workbook.Worksheets.get\_Item( 1 );

    Excel.Range range = worksheet.UsedRange;

    int nRows = 0;
    List<string> materials\_added = new List<string>();

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

          // Identity data of material class to duplicate.

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

            if( 0 < surPattern.Length
              && !surPattern.Equals( \_not\_available ) )
            {
              myMaterial.SurfacePatternId
                = fillPatterns[surPattern].Id;
            }

            if( 0 < cutPattern.Length
              && !cutPattern.Equals( \_not\_available ) )
            {
              myMaterial.CutPatternId
                = fillPatterns[cutPattern].Id;
            }
            materials\_added.Add( matName );
          }
        }
        ++nRows;
        ++iRow;
      }
      tx.Commit();
    }

    workbook.Close( true, null, null );
    excel.Quit();

    int n = materials\_added.Count;

    string msg = string.Format(
      "{0} row{1} successfully parsed and "
      + "{2} material{3} added:",
      nRows, PluralSuffix( nRows ),
      n, PluralSuffix( n ) );

    TaskDialog dlg = new TaskDialog(
      "Revit AddMaterials" );

    dlg.MainInstruction = msg;

    dlg.MainContent = string.Join( ", ",
      materials\_added ) + ".";

    dlg.Show();

    return Result.Succeeded;
  }
  catch( Exception ex )
  {
    message = "Revit AddMaterials Exception:\n"
      + ex.ToString();

    return Result.Failed;
  }
}
```

The code is actually shorter than all the explanations provided :-)

I hope these enhancements help:

- Understanding the system

- From the user point of view
- From the coding point of view

- Using the system

- As an end user utility
- Adding more enhancements as an add-in developer

For the complete source code, Visual Studio solution and add-in manifest, please refer to the
[AddMaterials GitHub repository](https://github.com/jeremytammik/AddMaterials).

The version discussed above is
[release 2014.0.0.1](https://github.com/jeremytammik/AddMaterials/releases/tag/2014.0.0.1).

Good luck!