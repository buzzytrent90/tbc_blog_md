---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.539209'
original_url: https://thebuildingcoder.typepad.com/blog/0201_dwg_export_filename.html
post_number: '0201'
reading_time_minutes: 3
series: general
slug: dwg_export_filename
source_file: 0201_dwg_export_filename.htm
tags:
- csharp
- elements
- revit-api
- selection
- vbnet
- views
title: DWG Export Filename
word_count: 638
---

### DWG Export Filename

Here is a question handled by Joe Ye that I thought might be of general interest.

It makes use of the Revit SDK sample ImportExport converted to VB.NET.
We mentioned the issue of converting Revit SDK samples from C# to VB in
[one post](http://thebuildingcoder.typepad.com/blog/2009/05/vb-samples-and-other-questions.html#2),
and Adam Nagy later presented a
[more complete port](http://thebuildingcoder.typepad.com/blog/2009/07/porting-from-c-to-vbnet.html).

**Question:**
I am making an application to export Revit views to DWG files.
I used the ImportExport sample converted to VB.NET as a basis.
I am having trouble defining filenames other than the built-in solution.
The syntax for defining the DWG filename for a given view seems to be:

```
[FileName] & " - " & [ViewType] & " - " & [ViewName]
```

The filename can be changed, but the other substrings seem to be hardcoded, or at least I can't find where they are defined.
I would like the FileName and ViewType substrings to be optional and maybe replaced by some other project defined prefix.
Is this possible through the API?

If not, a workaround could be to rename the DWG files afterwards, but I hope to avoid that.

**Answer:**
In this case, you are using one of the overloads of the Export method taking four arguments, the fourth of which specifies the type of export required.
The first three are folder, name, and views:

- folder: the output folder into which the files will be exported.- name: either the name of a single file or a prefix for a set of files. If null or empty, automatic naming will be used.- views: a selection of views to be exported. If null or empty, only the active view will be exported.

As the documentation says, the exported DWG file name is in fact generated using a hardcoded algorithm inside Revit if the view set contains more than one single element.

If only one view is specified, then the filename string constructed from three substrings provided by specific variables is used.
It is defined in the module ExportData.vb, in the Initialize function:

```vbnet
Private Sub Initialize()
  ' The directory into which the file will be exported
  Dim dllFilePath As String \_
    = Assembly.GetExecutingAssembly().Location

  m\_exportFolder = Path.GetDirectoryName(dllFilePath)

  ' The file name to be used by export
  Dim docTitle As String = m\_activeDoc.Title

  Dim v As View = m\_activeDoc.ActiveView

  Dim viewName As String = v.Name
  Dim viewType As String = v.ViewType.ToString()

  m\_exportFileName = docTitle \_
    + "-" + viewType \_
    + "-" + viewName \_
    + "." + getExtension().ToString()

  ' Whether current active view is 3D view

  If v.ViewType = Enums.ViewType.ThreeD Then
    m\_is3DView = True
  Else
    m\_is3DView = False
  End If
End Sub
```

Therefore, you can easily make FileName and ViewType optional or replace them by some other prefix.
Simply change the string values in the definition of m\_exportFileName as you please, for instance like this:

```csharp
m\_exportFileName = docTitle + "-" + aaaa + . + getExtension().ToString
```

Note that changes to the specified filename have no effect if more than one view is given in the third argument to the Export method.
If the third argument includes more than one view, Revit will automatically name each exported DWG file using a hard-coded rule including the second argument to the Export method as a prefix.
If the third argument includes just one single view, then the file name passed in is used.
So the workaround to use your own naming rule is to export only one view at a time.
In case of multiple views, you can export them one by one in a loop.

This information obviously also applies to the other overloads of the Export method taking a ViewSet argument, including export to:

- DGN- DWF2D- DWF3D- DWFX2D- DWFX3D- DWG- FBX

Thank you very much Joe for this answer!