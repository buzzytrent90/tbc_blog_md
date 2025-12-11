---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.8
content_type: qa
optimization_date: '2025-12-11T11:44:16.365070'
original_url: https://thebuildingcoder.typepad.com/blog/1597_curtain_wall_panel.html
post_number: '1597'
reading_time_minutes: 2
series: general
slug: curtain_wall_panel
source_file: 1597_curtain_wall_panel.md
tags:
- elements
- filtering
- geometry
- revit-api
- sheets
- walls
title: Curtain Wall Panel
word_count: 317
---

### Curtain Wall Panel Geometry with Basic Wall Panel
A quick geometrical question on retrieving geometry from a basic wall being used as a panel in a curtain wall:
\*\*Question:\*\* I am struggling to retrieve the geometry data from a curtain wall that contains a Basic wall in one of the curtain wall panels.
My example curtain wall has two panels. With one of the panels, a basic wall type is associated. I need to get the geometry data (i.e., the faces) for the entire curtain wall. When I reach the second panel in my code, the `SymbolGeometry` contains zero objects, so my code cannot retrieve any geometry for it. As a result, it is not able to handle the basic wall to retrieve its geometry face data.
![Curtain wall panel](img/curtain_wall_panel.png)
\*\*Answer:\*\* Here is a code snippet that handles this situation correctly.
The main point is this: we need to find the panel-wall which corresponds to the curtain wall and then retrieve its geometry in a second step:
```csharp
// First, find solid geometry from panel ids.
// Note that the panel which contains a basic
// wall has NO geometry!
Wall wall = doc.GetElement( curtainWallId ) as Wall;
var grid = wall.CurtainGrid;
foreach( ElementId id in grid.GetPanelIds() )
{
Element e = doc.GetElement( id );
solids.AddRange( GetElementSolids( e ) );
}
// Secondly, find corresponding panel wall
// for the curtain wall and retrieve the actual
// geometry from that.
FilteredElementCollector cwPanels
= new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_CurtainWallPanels )
.OfClass( typeof( Wall ) );
foreach( Wall cwp in cwPanels )
{
// Find panel wall belonging to this curtain wall
// and retrieve its geometry
if( cwp.StackedWallOwnerId == curtainWallId )
{
solids.AddRange( GetElementSolids( cwp ) );
}
}
```
I added this code in the method `GetCurtainWallPanelGeometry`
to [The Building Coder samples release 2018.0.134.6](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.134.6)
module [CmdCurtainWallGeom.cs L35-L66](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCurtainWallGeom.cs#L35-L66).