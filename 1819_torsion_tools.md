---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: qa
optimization_date: '2025-12-11T11:44:16.809491'
original_url: https://thebuildingcoder.typepad.com/blog/1819_torsion_tools.html
post_number: '1819'
reading_time_minutes: 4
series: general
slug: torsion_tools
source_file: 1819_torsion_tools.md
tags:
- geometry
- references
- revit-api
- schedules
- sheets
- views
title: Torsion Tools
word_count: 744
---

### Ini File, Localisation and Torsion Tools Two
More Revit API tutorial material and tools plus a couple of hints from the Revit API discussion forum and the Forge blog:
- [Torsion Tools two](#2)
- [Retrieve path to Revit.ini](#3)
- [Updated NeXLT localization URL](#4)
- [Volume and area of triangulated solid](#5)
#### Torsion Tools Two
Continuing right on from the
fruitful [collection of videos and tutorials](https://thebuildingcoder.typepad.com/blog/2020/02/search-for-getting-started-tutorials-and-videos.html) presented
yesterday, an update to
the [Torsion Tools add-in template with code examples for common tools](https://thebuildingcoder.typepad.com/blog/2020/01/torsion-tools-command-event-and-info-in-da4r.html) introduced last week:
- [Torsion Tools Update #2 – Copy Linked Legends, Schedules and Reference Views](https://youtu.be/C2dBRqXl9UA):
> Autodesk Revit 2020 API Visual Studio Solution Template with Code Examples for Common Tools.
Update #2 walks through the added tools to copy legends, schedules, and reference views from a linked model.
> The Copy Legends and Schedules tools allow you to select the Link to copy them from, then select the Legends or Schedules you would like to copy.
It will check whether they already exist in the project and prompt you if they do.
> The Linked Views tool allows you to select a linked model, title block, viewport type and then the views you would like from the linked model.
It creates a drafting view in the current document with the name, detail number and sheet of the linked model.
> [TorsionTools GitHub repo](https://github.com/TorsionTools/R20)

Many thanks to Torsion Tools for creating and sharing this powerful resource!
#### Retrieve Path to Revit.ini
A small note from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [adding project template to 'New Project' via API](https://forums.autodesk.com/t5/revit-api-forum/adding-project-template-to-new-project-via-api/m-p/8585348) by
Peter @cig_ad Ciganek:
> The \*Autodesk.Revit.ApplicationServices.Application\* property `CurrentUsersDataFolderPath` returns the path where Revit.ini is located.
#### Updated NeXLT Localization URL
In another small not from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on the [localization website](https://forums.autodesk.com/t5/revit-api-forum/localization-website/m-p/8500166),
Susan Renna points out the new URL:
> You can find the Autodesk NeXLT localization website here:
[ls-wst.autodesk.com/app/nexlt-plus/app/home/search](https://ls-wst.autodesk.com/app/nexlt-plus/app/home/search)
#### Volume and Area of Triangulated Solid
Finally, a useful generic pure geometry utility that might come in handy in the Revit environment as well
[to calculate the volume of a 3D mesh object, the surface of which is made up triangles](https://stackoverflow.com/questions/1406029/how-to-calculate-the-volume-of-a-3d-mesh-object-the-surface-of-which-is-made-up):
[Reading this paper](http://chenlab.ece.cornell.edu/Publication/Cha/icip01_Cha.pdf),
it is actually a pretty simple calculation.
The trick is to calculate the \*\*signed volume\*\* of a tetrahedron – based on your triangle and topped off at the origin.
The sign of the volume comes from whether your triangle is pointing in the direction of the origin
(The normal of the triangle is itself dependent upon the order of your vertices, which is why you don't see it explicitly referenced below).
This all boils down to the following simple function:

```
  public float SignedVolumeOfTriangle(Vector p1, Vector p2, Vector p3) {
    var v321 = p3.X*p2.Y*p1.Z;
    var v231 = p2.X*p3.Y*p1.Z;
    var v312 = p3.X*p1.Y*p2.Z;
    var v132 = p1.X*p3.Y*p2.Z;
    var v213 = p2.X*p1.Y*p3.Z;
    var v123 = p1.X*p2.Y*p3.Z;
    return (1.0f/6.0f)*(-v321 + v231 + v312 - v132 - v213 + v123);
  }
```

A driver uses it to calculate the volume of the mesh:

```
  public float VolumeOfMesh(Mesh mesh) {
    var vols = from t in mesh.Triangles
      select SignedVolumeOfTriangle(t.P1, t.P2, t.P3);
    return Math.Abs(vols.Sum());
  }
```

Adam Nagy picked this up and used it
to [calculate volume and surface area in the Forge viewer](https://forge.autodesk.com/blog/get-volume-and-surface-area-viewer),
verifying that the resulting values match the corresponding properties provided by Inventor:
![Volume and surface area in the Forge viewer](img/forge_viewer_volume_surface_area.png "Volume and surface area in the Forge viewer")