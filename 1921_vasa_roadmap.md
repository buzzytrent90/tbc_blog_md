---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:17.046798'
original_url: https://thebuildingcoder.typepad.com/blog/1921_vasa_roadmap.html
post_number: '1921'
reading_time_minutes: 6
series: general
slug: vasa_roadmap
source_file: 1921_vasa_roadmap.md
tags:
- elements
- geometry
- revit-api
- sheets
- transactions
- views
title: Vasa Roadmap
word_count: 1173
---

### Hiding Edges, AU, Roadmaps and Vasa
Numerous exciting announcements leading up to AU this week:
- [Revit roadmap update and AMA at AU](#2)
- [Structural news at AU](#3)
- [Revit category guide](#4)
- [How to hide internal edges of solids](#5)
- [VASA 3D voxel-based architectural space analysis](#6)
#### Revit Roadmap Update and AMA at AU
To provide an idea where Revit is headed and enable you to contribute and influence the plans,
[@AutodeskRevit](https://twitter.com/AutodeskRevit) shares
the [Revit Public Roadmap Update Fall 2021](https://blogs.autodesk.com/revit/2021/09/30/revit-public-roadmap-update-fall-2021):
> The [#Revit team](https://twitter.com/hashtag/Revit) is out with the latest edition of the Revit Public Roadmap.
Check it on the blog and don't forget to register
for [#AU21](https://twitter.com/hashtag/AU21),
where we're hosting [3 #AMA sessions](https://twitter.com/hashtag/AMA) (ask me anything) to
take your questions on what's new, what's in development, and the road ahead.
#### Structural News at AU
[Pawel Piechnik](https://twitter.com/piechnikp), Product Manager, Strategist and Structural Engineer standing at the intersection of AEC and IT industries, shares numerous other important recommendations for Revit Structural at AU, e.g.,
> Check out [this session](https://lnkd.in/e2SS9J7N) discussing an immersive, 'inside-#revit' structural design case study: "Revit-driven design of steel structures with real-time model updating using ENERCALC Structural Engineering Library design calculations.",
[BIM-Driven Engineering: Structural Design Without Redundant Workload – \*BES500004\*](https://events-platform.autodesk.com/event/autodesk-university-2021/planning/UGxhbm5pbmdfNjY2OTI4)
#### Revit Category Guide
Talking about roadmaps, here is a handy
[Revit Category Guide](https://docs.google.com/spreadsheets/d/1uNa77XYLjeN-1c63gsX6C5D5Pvn_3ZB4B0QMgPeloTw/edit#gid=1549586957), a spreadsheet listing all category names and associating Category Name, Built In Enum, User Mapped, Display Category Name, and Display Category Name in Russian.
#### How to Hide Internal Edges of Solids
Back to a pure programming topic, we gleaned some new information on how to hide internal edges of solids in a discussion on the appearance of `DirectShape` elements created with Dynamo vs directly using the Revit API:
\*\*Question:\*\* I made a custom node that creates volumes using `DesignScript.Geometry.Solid`.
If I convert this solid to a `DirectShape` in my node, the resulting solids obtained in Revit, `Revit.DB.Solid`, display the internal mesh edges.
If I use the Dynamo `DirectShape.ByGeometry` node instead, the resulting Revit is smooth and no internal mesh edges are shown.
![DirectShape tessellation edges](img/direct_shape_tessellation_edges.png "DirectShape tessellation edges")
Why?
How do I create a solid without mesh edges in my own code?
See my code for the creation of DirectShape:
```csharp
RevitDB.Material material = GetPassMaterial( doc );
using( RevitDB.Transaction t = new RevitDB.Transaction( doc,
"Create sphere direct shape" ) )
{
t.Start();
int sens = 1;
var sortedPointsVolumesDS = pointsVolumesDS.ToArray().OrderBy(
pv => origine.DistanceTo( pv.Key ) \* sens );
int passNumber = 1;
foreach( var item in sortedPointsVolumesDS )
{
string name = $"{fanName}_{driName}_{passNumber}";
// creation de la géometrie
IList revitGeometry = item.Value.ToRevitType(
RevitDB.TessellatedShapeBuilderTarget.Solid,
RevitDB.TessellatedShapeBuilderFallback.Abort,
material.Id );
//IList tessellatedShape = null;
// tessellatedShape = item.Value.ToRevitType(
// RevitDB.TessellatedShapeBuilderTarget.AnyGeometry,
// RevitDB.TessellatedShapeBuilderFallback.Abort,
// material.Id);
if( revitGeometry?.Count > 0 )
{
RevitDB.DirectShape ds = RevitDB.DirectShape.CreateElement( doc,
new RevitDB.ElementId( RevitDB.BuiltInCategory.OST_GenericModel ) );
ds.ApplicationId = name;
ds.ApplicationDataId = Guid.NewGuid().ToString(); // "Geometry object id";
ds.SetShape( revitGeometry );
//ds.SetShape(tessellatedShape);
}
// on incremente le numéro de la passe
passNumber++;
}
t.Commit();
}
```
\*\*Answer:\*\* The quick answer is simple:
The entire Dynamo framework is open source, giving you access to
the [full implementation source code](https://github.com/DynamoDS)
So, you might be able to find out how yourself from the Dynamo source code.
You could also discuss this in detail with the Dynamo experts in
the [Dynamo discussion forum](https://forum.dynamobim.com).
However, I have a suspicion or two of my own that I would like to check with the Revit development team for you first:
Maybe, the direct shape showing all the internal face tessellation edges has been defined using lots of separate individual triangular faces, whereas the other one uses just one single more complex planar face for the top and bottom.
Actually, I believe that direct shapes are not limited to linear edges, but can include curved edges as well. Maybe, the non-tessellated shape face does not consist of triangles at all, but just two straight edges and an arc.
Later: the development team confirm my hunch:
The Revit API doesn't have a way to turn on or off mesh edges.
However, creating a mesh through `TessellatedShapeBuilder` may attempt to hide certain edges depending on the topology of the mesh.
We have not yet changed or added any API functions for our recent project to hide (some) interior mesh edges.
We simply modified the function that marks certain mesh edges as hidden so that it marks all two-sided mesh edges as hidden.
Previously, it only marked (nearly) tangential edges as hidden.
It was already the case that `TessellatedShapeBuilder` called that function when it created a mesh.
However, `TessellatedShapeBuilder` may create a 'faceted' BRep, i.e., a BRep with planar faces and straight edges, instead of a mesh, in which case no edge hiding is done (as far as I know).
Moreover, the effectiveness of the new edge-hiding depends very much on the topological structure of the mesh, which itself depends on several factors.
Given that the sample code uses the option `TessellatedShapeBuilderTarget.Solid`, it could be that the Revit object is a `Solid` (internally, a BRep) and not a `Mesh` (internally, a GPolyMesh).
Dynamo is not hiding interior mesh edges.
It is simply creating a proper solid in this case, which it will do when it can.
Note that in the next major release, we will hide the mesh edges, although that will unfortunately fool the user into creating sub-optimal geometry as a result, creating a mesh, when a solid is possible.
In exploring this further, you might also appreciate this GitHub link to the open source Dynamo for Revit code base [DirectShape.cs module](https://github.com/DynamoDS/DynamoRevit/blob/5c3b0d869ccdc2f4d5fd24b5346933f22d39f279/src/Libraries/RevitNodes/Elements/DirectShape.cs).
Might not solve this problem entirely, but it should give a few strings to pull at.
To summarise: if the input geometry is a solid or surface, Dynamo tries to use the `BrepBuilder`; if it fails, it uses the tessellated shape builder, e.g., if the input geometry is a `Mesh`, then it uses the tessellated shape builder directly.
#### VASA 3D Voxel-Based Architectural Space Analysis
Talking about Dynamo, solids and volumes, you might be interested in Kean's presentation
of [VASA, a Dynamo tool for 3D voxel-based architectural space analysis](https://www.keanw.com/2021/09/introducing-vasa-for-voxel-based-architectural-space-analysis.html):
> Here’s a quick animation from the 'overview' sample that shows daylight, visibility and pathfinding combined. Performance is impressive – it’s very nearly interactive as we change the distance for the visibility field’s cut-off:
![VASA overview sample](img/vasa_overview_sample.gif "VASA overview sample")