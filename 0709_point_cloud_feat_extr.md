---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.2
content_type: news
optimization_date: '2025-12-11T11:44:14.440478'
original_url: https://thebuildingcoder.typepad.com/blog/0709_point_cloud_feat_extr.html
post_number: 0709
reading_time_minutes: 1
series: geometry
slug: point_cloud_feat_extr
source_file: 0709_point_cloud_feat_extr.htm
tags:
- elements
- geometry
- levels
- revit-api
- views
- walls
title: Point Cloud Feature Extraction
word_count: 254
---

### Point Cloud Feature Extraction

I discussed
[point cloud unit conversion](http://thebuildingcoder.typepad.com/blog/2012/01/point-cloud-unit-conversion.html) just
yesterday.
We can immediately follow up with some more hot news on point clouds:

The free technology preview of a
[point cloud feature extraction tool](http://labs.autodesk.com/utilities/scan_to_bim) for
Revit has been released on Autodesk Labs.
It allows you to work more easily with point clouds in Revit by automatically extracting useful geometry features from point clouds of buildings and creating basic Revit elements from them.

#### Tools Included

Point Cloud Feature Extraction for Autodesk Revit 2012 provides the following tools to facilitate the point cloud editing:

- Crop / Uncrop: Temporarily hide the points outside a rectangle or polygon- Hide Point Cloud: Temporarily hide the whole point cloud object to facilitate the inspection of feature extraction results- Adjust Axis: Transform the point cloud data so that a floor can be aligned with the XY plane and major walls are parallel to Z axis

Moreover, this plug-in includes some main features specifically for Revit so that the extracted features and geometry can be smoothly integrated into the BIM workflow:

- Datum Extraction: Extract both level and orthogonal grid- Site Extraction: Extract both terrain surface for ground surface creation and building footprint on terrain surface for building pad generation- Wall Extraction: Extract both straight wall layout and arc wall- Floor Extraction: Extract floor from selected points on the floor plan level

![Point Cloud](img/point_cloud_feature_extraction_crop.png)

Check it out on the labs!