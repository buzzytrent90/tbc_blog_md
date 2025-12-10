---
post_number: "0613"
title: "ImportExport Update"
slug: "importexport_update"
author: "Jeremy Tammik"
tags: ['csharp', 'parameters', 'references', 'revit-api', 'sheets', 'views', 'windows']
source_file: "0613_importexport_update.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0613_importexport_update.html"
---

### ImportExport Update

I mentioned some features of the ImportExport SDK sample in previous posts, e.g. on
[porting it from C# to VB](http://thebuildingcoder.typepad.com/blog/2009/07/porting-from-c-to-vbnet.html), setting the
[DWG export filename](http://thebuildingcoder.typepad.com/blog/2009/08/dwg-export-filename.html),
[modifying the DWF export filename](http://thebuildingcoder.typepad.com/blog/2009/12/modify-the-dwf-export-filename.html), and
[exporting a 3D view to 2D DWF](http://thebuildingcoder.typepad.com/blog/2010/02/export-3d-view-to-2d-dwf.html).

Now Kurt Schrempp ran into a little problem with this sample.
He says: I solved my problem.
Here is the issue:
When you try to export as DWG, it requires and gives you options to define the layer mapping.
If you don't open up the layer mapping options dialogue, no default settings are set, and a null argument exception is thrown.

Kurt provided a slightly updated version which fixes that and some other minor problems of the standard implementation.
The main fix is to automatically open the dialogue so that the default options are instantiated and set.
If this step is not performed, and the associated property is left set to its default value of null, the sample fails in Revit 2012.

In Kurt's words:
There are actually two errors due to similar things.
One is based on the layer options, the other on the selected views when selecting more than just the current view.

To reproduce the layer bug, launch ImportExport > Select Export > Select DWG format > Click OK > Select Directory > Click Save.

Now, because the options button is optional, and it is skipped by default, the error should come up.

To reproduce the view bug, launch ImportExport > Select Export > Select DWG format > Click OK > Select Directory > Select the "Selected views/sheets" Radio Button > Click Options > Click Ok > Click Save.

In this one, because the "..." button to select views is optional, no default views are set, similarly to the layer options issue, so a different error comes up.

Here is an image that illustrates a solution for this:

![ImportExport fix](img/KurtSchremppImportExportFix.png)

Here is the options button click handler implementation:
```csharp
private void buttonOptions\_Click(
  object sender,
  EventArgs e )
{
  // Export dwg
  if( m\_exportData.ExportFormat
    == ExportFormat.DWG )
  {
    bool contain3DView = false;

    if( radioButtonCurrentView.Checked )
    {
      if( m\_exportData.Is3DView )
      {
        contain3DView = true;
      }
    }
    else
    {
      if( m\_exportData.SelectViewsData
        .Contain3DView )
      {
        contain3DView = true;
      }
    }

    ExportDWGData exportDWGData
      = m\_exportData as ExportDWGData;

    using( ExportDWGOptionsForm exportOptionsForm
      = new ExportDWGOptionsForm(
        exportDWGData.ExportOptionsData,
        contain3DView ) )
    {
      exportOptionsForm.ShowDialog();
    }
  }
  // . . .
}
```

It sets up the DWGExportOptions parameter:
```csharp
  DWGExportOptions dwgExportOptions = new DWGExportOptions();
  dwgExportOptions.ExportingAreas = m\_exportOptionsData.ExportAreas;
  dwgExportOptions.ExportOfSolids = m\_exportOptionsData.ExportSolid;
  dwgExportOptions.FileVersion = m\_exportFileVersion;
  dwgExportOptions.LayerMapping = m\_exportOptionsData.ExportLayerMapping;
  dwgExportOptions.LineScaling = m\_exportOptionsData.ExportLineScaling;
  dwgExportOptions.MergedViews = m\_exportOptionsData.ExportMergeFiles;
  dwgExportOptions.PropOverrides = m\_exportOptionsData.ExportLayersAndProperties;
  dwgExportOptions.SharedCoords = m\_exportOptionsData.ExportCoorSystem;
  dwgExportOptions.TargetUnit = m\_exportOptionsData.ExportUnit;
```

These properties aren't set until this window initializes.
So unless you click the "Options" button on the previous form, the values are null, thus throwing the exception.

All I did was make it so the options form displays as soon as the export form comes up, setting the defaults immediately.

Here is the code of my [modified version of the ImportExport SDK sample](zip/ImportExportKs.zip) which opens up the two windows automatically, so there are no null defaults.

It implements a couple of other small changes as well.
Here is a list of the differing files:

- ExportData.cs – modify output filename- ExportDWGData.cs – rename output file- ExportWithViewsForm.cs – display layer mapping and options as discussed above- SelectViewsForm.cs – additional handling of 2D and 3D views

Many thanks to Kurt for these insights and sharing his update!

#### Avoid Casting Twice

Here is an additional little note on casting efficiency.

This code accesses the directory containing an external referenced file:
```csharp
  Type T = es.Current.GetType();
  if( T == typeof( ImportInstance ) )
  {
    ImportInstance imp = es.Current
      as ImportInstance;

    if( imp.IsLinked )
    {
      ExternalFileReference el = ExternalFileUtils
        .GetExternalFileReference(
          actdoc, imp.GetTypeId() );

      if( el != null )
      {
        directory = ModelPathUtils
          .ConvertModelPathToUserVisiblePath(
            el.GetAbsolutePath() );
      }
    }
    else
      error = "The import instance you selected"
        + " is not linked into the project.";
  }
```

It can be simplified by using the 'is' statement, replacing the first two lines by the following more readable statement:
```csharp
  if( es.Current is ImportInstance )
```

We can go one step further, though, because the next line is casting to an ImportInstance.
The sequence of statements saying 'if x is Y then: Z z = x as Y' is basically performing the cast twice over.
You can avoid that unneccessary cost by casting once only and testing the result agains null.
In this case, we can combine the first if statement with the cast and the second if statement, i.e. combine the line above with the one folowing it line and just say:
```csharp
  ImportInstance imp = es.Current
    as ImportInstance;

  if( null != imp && imp.IsLinked )
```

This eliminates the first two lines altogether, leaving just this:
```csharp
  ImportInstance imp = es.Current
    as ImportInstance;

  if( null != imp && imp.IsLinked )
  {
    ExternalFileReference el = ExternalFileUtils
      .GetExternalFileReference(
        actdoc, imp.GetTypeId() );

    if( el != null )
    {
      directory = ModelPathUtils
        .ConvertModelPathToUserVisiblePath(
          el.GetAbsolutePath() );
    }
  }
  else
  {
    error = "The import instance you selected"
      + " is not linked into the project.";
  }
```

For more details, you can refer to sections 8.2.2 'The is Operator', 8.2.3 'The as Operator', and especially 8.2.4 'The is Operator Versus the as Operator' in this
[C# tutorial](http://etutorials.org/Programming/Programming+C.Sharp/Part+I+The+C+Language/Chapter+8.+Interfaces/8.2+Accessing+Interface+Methods).