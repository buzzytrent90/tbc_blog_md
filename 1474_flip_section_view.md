---
post_number: "1474"
title: "The Building Coder"
slug: "flip_section_view"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'references', 'revit-api', 'rooms', 'sheets', 'views']
source_file: "1474_flip_section_view.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1474_flip_section_view.html"
---

### Flipping a Section View and Forge Webinar 3
I already created a starting point for the
new [roomedit3d](https://github.com/jeremytammik/roomedit3d) incarnation
last week, in its own
repository [roomedit3dv3](https://github.com/jeremytammik/roomedit3dv3).
However, please forget that right away.
This week, I agreed with Philippe to retain the new project in an own
dedicated [roomedit3d branch](https://github.com/Autodesk-Forge/forge-boilers.nodejs/tree/roomedit3d) within his
original [Forge boilerplate code sample collection](https://github.com/Autodesk-Forge/forge-boilers.nodejs).
Hopefully, that will simplify keeping my BIM sample synchronised with any future updates make to that.
Now I need to get started implementing and testing the actual viewer extension functionality itself.
My main topic today is something different and purely Revit API oriented, besides the next Forge session:
- [Flipping a section view](#2)
- [Forge webinar series](#3)
#### Flipping a Section View
Here is a recent interesting little case handled by Jim Jia and answered by Paolo Serra:
\*\*Question:\*\* I want to create a new model with a section view based on an existing one with the same cross-sectional view but a different crop box.
To do so, I perform, these steps:
1. Open the old model file, find the required cross-sectional view.
2. Create a new model within the same bounding box, using the old model view CropBox created like this:
```csharp
Transform transform = view.CropBox.Transform;
BoundingBoxXYZ box = new BoundingBoxXYZ();
box.Enabled = true;
XYZ maxPoint = view.CropBox.Max;
XYZ minPoint = view.CropBox.Min;
box.Transform = transform;
box.Max = maxPoint;
box.Min = minPoint;
ViewSection viewSection = ViewSection.CreateSection(
doc, view.GetTypeId(), box );
```
This works and I can create a cross-sectional view through this method, but in the opposite direction of the existing view, like this:
![Section view flipped](img/viewsection_flipped.png)
How can I flip the view direction to solve this problem?
\*\*Answer:\*\* A few days ago I was trying to do the same.
What worked for me was to reverse the `RightDirection` vector of the original view and recreate a transform to apply to the `BoundingBoxXYZ`.
You should also take into consideration the different transforms you might have in the two documents.
```csharp
Transform tr = view.CropBox.Transform;
tr.BasisX = -view.RightDirection;
tr.BasisY = XYZ.BasisZ;
tr.BasisZ = tr.BasisX.CrossProduct( tr.BasisY );
BoundingBoxXYZ bb = new BoundingBoxXYZ();
bb.Transform = tr;
XYZ min = view.CropBox.Min;
XYZ max = view.CropBox.Max;
bb.Min = new XYZ( min.X, min.Y, 0 );
bb.Max = new XYZ( max.X, max.Y, -min.Z );
ElementId vftId = new FilteredElementCollector( doc )
.OfClass( typeof( ViewFamilyType ) )
.WhereElementIsElementType()
.Cast()
.First( x => x.ViewFamily == ViewFamily.Section )
.Id;
View newView = ViewSection.CreateSection( doc, vftId, bb );
```
\*\*Response:\*\* Thanks a lot, it works!
Many thanks to Jim and Paolo for sharing this!
#### Forge Webinar Series
Tomorrow we are presenting the third session in the ongoing Forge webinar series, on the Model Derivative API.
Support documentation and recordings from the first two sessions for your future reference and enjoyment:
- September 20 [Introduction to Autodesk Forge and the Autodesk App Store](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-autodesk-forge-and-the-autodesk-app-store.html)
- September 22 [Introduction to OAuth and Data Management API](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-oauth-and-data-management-api.html)
– on [OAuth](https://developer.autodesk.com/en/docs/oauth/v2/overview)
and [Data Management API](https://developer.autodesk.com/en/docs/data/v2/overview), providing token-based authentication, authorization and a unified and consistent way to access data across A360, Fusion 360, and the Object Storage Service.
Upcoming sessions continuing during the remainder of
the [Autodesk App Store Forge and Fusion 360 Hackathon](http://autodeskforge.devpost.com) until the end of October:
- September 27 – [Model Derivative API](https://developer.autodesk.com/en/docs/model-derivative/v2/overview) – enable users to represent and share their designs in different formats and extract metadata.
- September 29 – [Viewer API](https://developer.autodesk.com/en/docs/viewer/v2/overview) –
formerly part of the 'View and Data API', a WebGL-based, JavaScript library for 3D and 2D model rendering a CAD data from seed models, e.g., [AutoCAD](http://www.autodesk.com/products/autocad/overview), [Fusion 360](http://www.autodesk.com/products/fusion-360/overview), [Revit](http://www.autodesk.com/products/revit-family/overview), and many other formats.
- October 4 – [Design Automation API](https://developer.autodesk.com/en/docs/design-automation/v2/overview) – formerly known as 'AutoCAD I/O', run scripts on design files.
- October 6 – [BIM360](https://developer.autodesk.com/en/docs/bim360/v1/overview) – develop apps that integrate with BIM 360 to extend its capabilities in the construction ecosystem.
- October 11 – [Fusion 360 Client API](http://help.autodesk.com/view/NINVFUS/ENU/?guid=GUID-A92A4B10-3781-4925-94C6-47DA85A4F65A) – an integrated CAD, CAM, and CAE tool for product development, built for the new ways products are designed and made.
- October 13 – Q&A on all APIs.
- October 20 – Q&A on all APIs.
- October 27 – Submitting a web service app to Autodesk App store.
![Forge – build the future of making things together](img/forge_accelerator.png)