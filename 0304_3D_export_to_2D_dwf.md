---
post_number: "0304"
title: "Export 3D View to 2D DWF"
slug: "3D_export_to_2D_dwf"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'sheets', 'views']
source_file: "0304_3D_export_to_2D_dwf.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0304_3D_export_to_2D_dwf.html"
---

### Export 3D View to 2D DWF

By the time you read this, I will already be away on my holiday in Andalusia.
Still, I thought I could drop off this last post before I leave.

We looked at various aspects of DWF export in the past, such as the
[unique id](http://thebuildingcoder.typepad.com/blog/2009/02/uniqueid-dwf-and-ifc-guid.html) assigned to elements, the
[view definition](http://thebuildingcoder.typepad.com/blog/2009/04/dwf-view-definition.html), and the
[export filenames](http://thebuildingcoder.typepad.com/blog/2009/12/modify-the-dwf-export-filename.html) used.

Here is another quick question on DWF export handled by Saikat Bhattacharya that may be of general interest:

**Question:** Is it possible to use the Revit API to export a 3D view in Revit to a 2D page in the generated DWF file?
I tried using both DWFX2DExportOptions and DWF2DExportOptions, and both generate a 3D DWF when I export a 3D view from Revit.

**Answer:** When working from the Revit user interface, 3D views are always exported to 3D DWF files, and plans and elevations as sheets in 2D DWF ones.
As usual in Revit, the functionality provided by the API parallels the product functionality, so I do not see a way to export 3D Revit models into 2D DWF sheets using the API.
As a quick test, you can play around with the Revit SDK ImportExport sample, where you can select a 3D view and choose 2D DWF as the output format.
You still get a 3D representation of the model in DWF, and not a 2D sheet.
As a workaround, though, you can always create a sheet in the Revit model containing the 2D representation of the 3D view.
If you export this sheet into DWF, the resulting file will represent the 3D view as a 2D sheet in DWF format.
Here is an example of using this workaround:

![2D DWF sheet displaying 3D Revit View](img/3D_export_to_2D_dwf.png)

Many thanks to Saikat for this solution!