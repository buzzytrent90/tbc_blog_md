---
post_number: "1871"
title: "Elbow Centre"
slug: "elbow_centre"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'revit-api', 'rooms', 'sheets']
source_file: "1871_elbow_centre.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1871_elbow_centre.html"
---

### FireRevit, Deprecated API and Elbow Centre Point
Struggling to find time to blog between cases, here is today's ration:
- [Revit 2021 `DisplayUnitType`](#2)
- [Eliminated TBC samples deprecated API usage](#3)
- [Calculating the elbow centre](#4)
- [FireRevit identifies room location for fire escape routes](#5)
#### Revit 2021 DisplayUnitType
Stephen Harrison raised and solved an issue on handling
[Revit 2021 `DisplayUnitType`](https://forums.autodesk.com/t5/revit-api-forum/revit-2021-displayunittype/m-p/9810626):
\*\*Question:\*\* I am struggling to update a section of my code I have been utilising for a few years now.
In Revit 2021, it is causing a deprecated API usage warning, so I need to update it:
- warning CS0618: `DisplayUnitType` is obsolete:
This enumeration is deprecated in Revit 2021 and may be removed in a future version of Revit.
Please use the `ForgeTypeId` class instead.
Use constant members of the `UnitTypeId` class to replace uses of specific values of this enumeration.
My code snippet is:
```csharp
case StorageType.Double:
double? nullable = t.AsDouble( fp );
if( nullable.HasValue )
{
DisplayUnitType displayUnitType = fp.DisplayUnitType;
value = UnitUtils.ConvertFromInternalUnits(
nullable.Value, displayUnitType ).ToString();
break;
}
```
Note: `t` is a `FamilyType` and `fp` is a `FamilyParameter` object.
\*\*Answer:\*\* Embarrassing how simple the solution was:
```csharp
// Pre 2021
DisplayUnitType displayUnitType = fp.DisplayUnitType;
value = UnitUtils.ConvertFromInternalUnits(
nullable.Value, displayUnitType ).ToString();
//2021
ForgeTypeId forgeTypeId = fp.GetUnitTypeId();
value = UnitUtils.ConvertFromInternalUnits(
nullable.Value, forgeTypeId ).ToString();
```
Many thanks to Stephen for sharing this!
#### Eliminated TBC Samples Deprecated API Usage
Stephen's question and answer prompted me to take another look at and try to eliminate the deprecated API usage warnings
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).
After the initial [flat migration to Revit 2021](https://thebuildingcoder.typepad.com/blog/2020/04/2021-migration-add-in-language-and-bim360-login.html#4),
the compilation still generates [162 warnings](zip/tbc_samples_2021_migr_01.txt),
all associated with deprecated methods and enumerations caused by
the [Units API changes](https://thebuildingcoder.typepad.com/blog/2020/04/whats-new-in-the-revit-2021-api.html#4.1.3).
The resolution was actually quite simple.
I removed some sections of code completely that dealt exclusively with the Revit Unit API functionality, since they ought to be rewritten from scratch for the new unit handling methods. It would make little sense to migrate them step by step. That left a handful of trivial issues to fix.
The new [release 2021.0.150.6](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2021.0.150.6) compiles with zero errors and warnings.
You can see exactly how that was achieved by analysing
the [differences to the previous release](https://github.com/jeremytammik/the_building_coder_samples/compare/2021.0.150.5...2021.0.150.6).
#### Calculating the Elbow Centre
From
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to calculate the centre point of an elbow](https://forums.autodesk.com/t5/revit-api-forum/how-to-calculate-the-center-point-of-elbow/m-p/9804339):
\*\*Question:\*\* How to calculate the centre point of elbow?
I am trying to get to centre point of an elbow.
I have checked all the data of the elbow in RevitLookup but did not find anything.
So, is there any way I can get that information?
![Elbow arc centre point](img/elbow_arc_centre_point_1.png "Elbow arc centre point")
\*\*Answer:\*\* Yes.
This is a pretty simple geometrical exercise.
You have the full information about the start and end point, specifically including location and direction == face normal.
The start and end point directions are not parallel. Therefore, they define a 2D plane. The centre point will normally lie in that plane, so you just have a (simpler) 2D task to solve, not 3D.
In the 2D plane, you can determine the normal vector to the start and end directions. Extend those two normal vectors to define infinite lines and determine their intersection. That is your centre point.
\*\*Response:\*\* Thank you for your answer.
I just found out that actually the elbow already has the centre point information under its Geometry information:
![Snooping elbow arc centre point](img/elbow_arc_centre_point_2.png "Snooping elbow arc centre point")
So, my working code is:
```csharp
static public XYZ GetCenterofElbow(
FamilyInstance selectedDuct )
{
XYZ output = null;
List allConnectors = selectedDuct.MEPModel
.ConnectorManager.Connectors
.Cast().ToList();
Connector connectorA = allConnectors[ 0 ];
Connector connectorB = allConnectors[ 0 ];
GeometryElement geometryElement = selectedDuct
.get_Geometry( new Options() );
List ginsList = selectedDuct
.get_Geometry( new Options() )
.Where( o => o is GeometryInstance )
.Cast()
.ToList();
foreach( GeometryInstance gins in ginsList )
{
foreach( GeometryObject ge in gins.GetInstanceGeometry() )
{
try
{
Arc centerArc = ge as Arc;
output = centerArc.Center;
}
catch( Exception )
{
}
}
}
return output;
}
```
\*\*Answer:\*\* Thank you for letting us know that simple solution!
Oh dear. I wasted some effort, then.
I implemented a geometrical solution for you based on the elbow connectors and their coordinate systems:
- [Connector Origin](https://www.revitapidocs.com/2020/28a9cf5e-9191-f9ce-74c8-622a681201f6.htm)
- [CoordinateSystem property returning a Transform object](https://www.revitapidocs.com/2020/cb6d725d-654a-f6f3-fed0-96cc618a42f1.htm)
- [New `GetElbowConnectors` and `GetElbowCentre` methods in The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdRectDuctCorners.cs#L240-L319)
```csharp
///
/// Return elbow connectors.
/// Return null if the given element is not a
/// family instance with exactly two connectors.
/// summary>
List GetElbowConnectors( Element e )
{
List cons = null;
FamilyInstance fi = e as FamilyInstance;
if( null != fi )
{
MEPModel m = fi.MEPModel;
if( null != m )
{
ConnectorManager cm = m.ConnectorManager;
if( null != cm )
{
ConnectorSet cs = cm.Connectors;
if( 2 == cs.Size )
{
cons = new List( 2 );
bool first = true;
foreach( Connector c in cs )
{
if( first )
{
cons[0] = c;
}
else
{
cons[1] = c;
}
}
}
}
}
}
return cons;
}
///
/// Return elbow centre point.
/// Return null if the start and end points
/// and direction vectors are not all coplanar.
/// summary>
XYZ GetElbowCentre( Element e )
{
XYZ pc = null;
List cons = GetElbowConnectors( e );
if( null != cons )
{
// Get start and end point and direction
XYZ ps = cons[ 0 ].CoordinateSystem.Origin;
XYZ vs = cons[ 0 ].CoordinateSystem.BasisZ;
XYZ pe = cons[ 1 ].CoordinateSystem.Origin;
XYZ ve = cons[ 1 ].CoordinateSystem.BasisZ;
XYZ vd = pe - ps;
// For a regular elbow, Z vector is normal
// of the 2D plane spanned by the coplanar
// start and end points and direction vectors.
XYZ vz = vs.CrossProduct( vd );
if( !vz.IsZeroLength() )
{
XYZ vxs = vs.CrossProduct( vz );
XYZ vxe = ve.CrossProduct( vz );
pc = Util.LineLineIntersection(
ps, vxs, pe, vxe );
}
}
return pc;
}
```
Would you like to test my code and see whether it returns the same result?
By the way, some elbow families have no arc segment at all.
For example, some can consist of two 45-degree segments, connected by a cylindrical part in between.
In this case, there would be two arcs with different centre points, so the connector based approach seems more flexible for different family content.
For example, here is a typical [German chimney exhaust elbow](https://www.ofenseite.com/1020147-pelletofenrohr-90-bogen-mit-kesselanschluss-muffe)
![Chimney 90 degree elbow](img/elbow_arc_centre_point_3.jpg "Chimney 90 degree elbow")
In this case, there isn't any arc at all.
The connector-based code should solve the task for that as well.
Please confirm that it works.
The code is untested.
#### FireRevit Identifies Room Location for Fire Escape Routes
[Luhan Neo Sheng](mailto:Neo Sheng ) shared a research project with potential practical applications in
his [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [FireRevit: Using Revit files to identify the room locations of fires and escape routes](https://forums.autodesk.com/t5/revit-api-forum/firerevit-using-revit-files-to-identify-the-room-locations-of/td-p/9809450):
FireRevit is software that parses Revit files in order to map from a location given as (latitude, longitude, height) to a Revit room identifier.
FireRevit also combines the Revit file and information about which rooms are inaccessible in an emergency to guide residents to the nearest exit by giving a route map.
Our application use case is firefighting where drones identify the locations of rooms having fires both to direct firefighters to rooms on fire and to help residents escape.
We believe this could be useful in general for Revit users.
Here is [our technical report RevitToDatabase.pdf](https://cs.nyu.edu/media/publications/RevitToDatabase.pdf)
([local copy](zip/firerevit_room_location.pdf)) and
the source code in the [LuhanSheng Revit_To_Database GitHub repository](https://github.com/LuhanSheng/Revit_To_Database).
Noteworthy related topics:
- Kean Walmsley implemented a [HoloLens-based tool for navigating low visibility environments and following an exit path](https://thebuildingcoder.typepad.com/blog/2016/09/hololens-escape-path-waypoint-json-exporter.html)
- Revit 2020 enhanced the Analysis API with support for a [Path of Travel API](https://thebuildingcoder.typepad.com/blog/2019/04/whats-new-in-the-revit-2020-api.html#4.2.19)
- The Revit SDK includes a [PathOfTravel SDK sample](https://thebuildingcoder.typepad.com/blog/2019/04/new-revit-2020-sdk-samples.html#5)