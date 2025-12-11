---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.390035'
original_url: https://thebuildingcoder.typepad.com/blog/0116_dwf_view_definition.html
post_number: '0116'
reading_time_minutes: 3
series: views
slug: dwf_view_definition
source_file: 0116_dwf_view_definition.htm
tags:
- parameters
- revit-api
- views
title: DWF View Definition
word_count: 521
---

### DWF View Definition

Here is a question is related to the 3D views in Revit, i.e. standard, top, left, bottom, etc. and user created, and the corresponding ones in a DWF file.
The issue concerns the relationship between the internal Revit coordinate system and the exported coordinates in DWF.
For instance, how to create a 3D DWF with a custom view using the coordinates of the 3D view defined in Revit.

In a DWF file, the view is determined by

- Camera position
- Target toward which the camera is pointing
- Up vector
- Width
- Height

Here is a description of these parameters from the
[HOOPS/3dGS Programming Guide](http://developer.hoops3d.com/documentation/Hoops3DGS/prog_guide/03_2_viewing_modelling_cameras.html).
These parameters are defined for the standard views in one of the XML files contained in the DWF file.

In Revit, the view is determined by the ViewDirection, UpDirection, and EyePosition properties of the View3D class in the Revit API.
When exporting a Revit file to DWF in Revit Architecture 2009, the parameters of the top view in the DWF file, for example (position, target, up vector), are different from the top view properties in Revit.
Here are the approximate values of the top view parameters extracted from the Revit file and its corresponding DWF file:

Revit top view:

- Eye position: 16.47, 7.45, 31.58
- Up direction: 0, 1, 0
- View direction: 0, 0, 1

DWF top view:

- Position: 10, 15.01, -7.5
- Target: 10, 0, -7.5
- Up: 0, 0, -1
- Field width: 48.045
- Field height: 48.045

**Question:**
What transformation should I apply to the 3D coordinates of view parameters in Revit so that I can get the corresponding 3D coordinates in DWF?
When I create custom views in Revit I want to make them available and apply them in the corresponding DWF file as well.
I also need to calculate the field width and height parameters of the DWF view, which do not have analogues in the Revit 3D view.

**Answer:**
Here are the transformations used to calculate the parameters of a DWF view from the Revit view.
The camera for the default view and named views are defined slightly differently.

This is the definition for the default view:

- DWF Position = Rvt\_Position.x, Rvt\_Position.z, -Rvt\_Position.y;
- DWF Up Vector = Rvt\_UpVector.x, Rvt\_UpVector.z, -Rvt\_UpVector.y;
- DWF Target = Rvt\_TargetPos.x, Rvt\_TargetPos.z, -Rvt\_TargetPos.y;
- DWF Field of View = Rvt\_FieldOfView.Width, Rvt\_FieldOfView.Height;

This is the one for a named view:

- unitScale = recorded in DWF as part of the transformation;
- DWF Position = Rvt\_Position.x / unitScale, Rvt\_Position.z / unitScale, -Rvt\_Position.y / unitScale;
- DWF Up Vector = Rvt\_UpVector.x / unitScale, Rvt\_UpVector.z / unitScale, -Rvt\_UpVector.y / unitScale;
- DWF Target = Rvt\_TargetPos.x / unitScale, Rvt\_TargetPos.z / unitScale, -Rvt\_TargetPos.y / unitScale;
- DWF Field of View = Rvt\_FieldOfView.Width / unitScale, Rvt\_FieldOfView.Height / unitScale;

Actually, the transformation for the default view is just a special case of the named view one with the unit scale set to one.

Many thanks to Joe Ye for handling this case!