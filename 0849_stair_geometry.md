---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: qa
optimization_date: '2025-12-11T11:44:14.730911'
original_url: https://thebuildingcoder.typepad.com/blog/0849_stair_geometry.html
post_number: 0849
reading_time_minutes: 6
series: geometry
slug: stair_geometry
source_file: 0849_stair_geometry.htm
tags:
- elements
- family
- geometry
- levels
- parameters
- revit-api
- rooms
- views
- walls
title: Stair Geometry and Roombook
word_count: 1136
---

### Stair Geometry and Roombook

A long time back we touched on the difficulties and challenges in retrieving
[column and stair geometry](http://thebuildingcoder.typepad.com/blog/2010/02/retrieving-column-and-stair-geometry.html).

From Revit 2013 onwards, you have full API access to create and read data from stairs and railings, as well as their separate subcomponents, as you can see from this excerpt of the What's New section in the Revit API help file RevitAPI.chm:

#### Stairs and Railings API – Stairs and Stair Components

The new Stairs and StairsType classes in the Autodesk.Revit.DB.Architecture namespace provide access to new stairs elements in the Revit database. Note that these API classes only provide access to the Stairs created 'by component' introduced in this release.
Stair elements created by sketch cannot be accessed as a Stairs object in the API.
It is possible to identify both types of stairs using the BuiltInCategory OST\_Stairs. The static predicate method Stairs.IsByComponent
indicates whether a stair alement is created by components (runs, landings and supports) or not.

The classes StairsLanding, StairsLandingType, StairsRun and StairsRunType provide access to the subcomponents and subcomponent types of the new Stairs elements.

The pre-2013 stairs are still supported by Revit in old models, and their geometry is still not accessible.

Several useful hints about pre-2013 stair geometry access can be gleaned from the Autodesk Revit API forum discussion on
[accessing stair geometry](http://forums.autodesk.com/t5/Autodesk-Revit-API/How-to-get-information-about-the-stairs-using-Revit-API/td-p/3420589),
so I edited and summarised it a bit for reproduction here:

**Question:** I have a multiple floor building with stairs and need to access the stair geometry using the Revit API.

However, it returns empty values for stairs.

What can we do?

**Answer:** Stairs are not exposed in the Revit API prior to 2013.
Please look at the
[Revit 2013 API overview](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#4) and
[SDK update](http://thebuildingcoder.typepad.com/blog/2012/04/developer-center-and-sdk-update.html#23).

Unfortunately, only post-2013 stairs provide access to full location and geometry information.

For pre-2013 stairs, you will have to use other means.

**Question:** My stair models were created in Revit 2011 and 2012.
I am trying to retrieve their information using the Revit 2013 API.

I can access information like Name, TypeName, FamilyName, TopLevel, BaseLevel, ActualNumRiser, ActualRiserHeight, TreadWidth, TreadDepth, TravelDistance, etc.

What I actually is geometry information like location points.

Is there any method to get the location information and the rooms associated with stairs?

**Answer**: You could calculate location points for pre-2013 stairs in a variety of different ways.

First, you could get the BoundingBoxXYZ of your stair.
Using this, you could calculate its lower-most centre point as your desired location point.

Second, you could browse the stair geometry, also searching for the bottom-most point.
For this, you need to iterate over all its solids and faces.

Third, there is a Sketch assigned to non-FamilyInstance based stairs.
You can access it using
[unsupported element id relationships](http://thebuildingcoder.typepad.com/blog/2011/11/undocumented-elementid-relationships.html).

Having calculated a proper point, you can use the Document GetRoomAtPoint method to find the proper room.

You might be able to compare the elevations of your levels to find the one fitting your location point, but actually both sketch and family based stairs already maintain a built-in parameter STAIRS\_BASE\_LEVEL\_PARAM which provides the level directly with no need to iterate and search through all level elevations.

**Question:** By using BoundingBoxXYZ I can only get the Max and Min points of the stair bounding box.
I still do not know location point of the landing or floor at the top and bottom of each stairway.
How can I get these points from the bounding box?

Also, GetRoomAtPoint returns null when I try to use it to retrieve the room name.
What should be causing that?

**Answer**: GetRoomAtPoint needs a point that is inside the room volume.
It may be useful to ensure that the point you test is not located directly on the volume border.
Because of tolerance issues, it could easily end up being falsely calculated as very slightly outside.

LocationPoint from BoundingBoxXYZ:
What I meant was that you could calculate the point that is in the lower centre of the bounding box, meaning on the bottom.
Add one meter to its Z value, e.g., to ensure it to be inside the 3D room volume.
Use that to calculate a point to be passed to GetRoomAtPoint.

To calculate the landing and floor location points, you could analyze the stair geometry, meaning that you need to interpret it.

**Question:** Is it possible to analyze the stair's geometry for the Revit file created prior to 2013?

And is it possible to access the geometry information of stairs created by sketch and not by component?

**Answer** As I wrote in the comments on
[unsupported element id relationships](http://thebuildingcoder.typepad.com/blog/2011/11/undocumented-elementid-relationships.html):

"But even if we cannot \*modify\* a stair this way, we can get \*more\* information than we would get if we just read the stair's parameters or properties.

For example, the longest CurveArray in the Sketch.Profile of a stair can be combined with the number of steps to calculate the 3D data of each step's front side.
The curves in the profile won't give us the correct Z data because the sketch is just a projection, but using the stair height and number of steps, we can calculate the Z distance for each step."

Of course you only can obtain 'stupid' geometry such as lines, faces and solids.

By combining that with the parameter values you mention above, you could \*calculate\* the information you want.

I must admit that there is no common approach for each type of stairs, but the point is how to obtain any information \*at all\*.

Many thanks to Rudolf the Revitalizer for this deep and inspiring explanation!

#### Revit 2013 Roombook Extension

The Revit Roombook extension helps calculate the surface area of walls, floors and ceiling elements, room circumferences and the total number of furnishing elements.
It enables automated detection of room areas and surfaces and supports users in configuring this data to local requirements, achieving more accurate model take-offs, and exporting quantified results to Excel and QTO, Autodesk Quantity Takeoff.

I happened to notice this sweet
[15 minute video](http://www.youtube.com/watch?v=6TPaF_ALAC0) explaining
it in full detail:

A special highlight that I particularly enjoyed in this case is the effective and pleasant narration in a soft, very nicely and slightly accented female voice :-)