---
post_number: "0015"
title: "RoomsRoofs Sample"
slug: "roofsrooms"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'revit-api', 'rooms', 'views']
source_file: "0015_roofsrooms.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0015_roofsrooms.html"
---

### RoomsRoofs Sample

Remaining on the track of geometrical topics, for the time being...

A common inquiry is how to determine the roof part of the boundary of a given room or space. Currently, Revit does not expose such an API, and there is no straightforward way to obtain this information. However, the new Revit SDK sample RoomsRoofs is provided to answer this question. It demonstrates how to check whether a room or space has a bounding roof, and determines all roof elements bounding a room or space from above.

Note that no new API functionality is used in this sample. The required calculations are done through a purely geometrical analysis. The sample queries the roof and the room or space for its geometry and then checks for intersections between the various elements. If the room or space intersects with a roof, it retrieves the room or space closed shell as a GeometryObjectArray, and extracts every solid element from it. It also determines the roof solid, and then feeds the two solids into the method AreSolidsCut( Solid, Solid ), which checks whether any of the solid faces overlap each other. You can also make use of AreSolidsCut() as a helper method for your own applications. The sample is not guaranteed to work with every possible configuration, so you may need to enhance it further for special cases.

The new sample is included in the standard Revit SDK distribution. It is currently also available for download from the ADN site in the Revit knowledgebase section under
[Revit 2009 Samples and Documents](http://adn.autodesk.com/adn/servlet/item?siteID=4814862&id=11234791&linkID=4901650).
For your convenience, I am also providing a copy of it right
**here**.

It works in Revit Architecture, in which case the analysis is performed for rooms, and for Revit MEP, in which case spaces can be analysed, and is written in C#. Separate sample files are provided for these two situations. Here is the 3D view of the architectural sample file:

![RoofsRooms sample file in 3D view](img/roofs_rooms_3d.png)

Here is the architectural sample file in plan view:

![RoofsRooms sample file in plan view](img/roofs_rooms_plan.png)

To run the sample, simply execute the external command. It determines all the rooms and spaces, calculates their roofs, and lists the analysis result in a dialogue box.

![RoofsRooms analysis results](img/roofs_rooms_message_box.png)