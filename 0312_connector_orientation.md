---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: qa
optimization_date: '2025-12-11T11:44:13.726989'
original_url: https://thebuildingcoder.typepad.com/blog/0312_connector_orientation.html
post_number: '0312'
reading_time_minutes: 1
series: mep
slug: connector_orientation
source_file: 0312_connector_orientation.htm
tags:
- elements
- family
- revit-api
- views
- mep
title: Connector Orientation
word_count: 287
---

### Connector Orientation

I am still looking through all the interesting cases my colleagues dealt with during my absence.
Here is another one handled by Joe Ye on the width and height orientation of a Revit MEP connector:

**Question:** Given an Autodesk.Revit.DB.Connector object, is it possible to know where the height and width dimensions are relative to the Connector.CoordinateSystem?
The width and height directions do seem to be aligned to either the coordinate system X or Y axis, but not consistently one way or the other.
I need a way of knowing which axis of the coordinate system the width and height align with.

**Answer:** The width is relative to the x axis and the height is relative to the y axis of the coordinate system of the connector.
There is a difference between the width and height of the connector and the width and height that is exposed in the user interface options bar.
The UI is based upon the orientation of the element in the current view.
The normal of the MEP element's underlying curve object and the family instance Z axis is used to determine this.
Extra checks need to be made for fittings based upon the z axis of the family, because they can be oriented not only relative to plan.
Typically, if the family's z axis is 0,0,1, you can assume the width and height is correct.
If it is not, the width and height need to be swapped when the rotation is greater than 45 degrees from vertical.
The connector is bound to the face of the family, so it does not rotate independently.

Many thanks to Harry Mattison and Joe for this explanation!