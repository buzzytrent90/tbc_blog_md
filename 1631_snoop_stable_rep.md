---
post_number: "1631"
title: "Snoop Stable Rep"
slug: "snoop_stable_rep"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'references', 'revit-api', 'selection', 'sheets', 'views', 'walls']
source_file: "1631_snoop_stable_rep.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1631_snoop_stable_rep.html"
---

### Export Geometry and Snoop Stable Representation
Александр Пекшев aka Modis [@Pekshev](https://github.com/Pekshev) submitted
a very succinct and useful pull request for [RevitLookup](https://github.com/jeremytammik/RevitLookup) that
I integrated right away, and provides many other valuable inputs as well:
- [Snoop stable representation of References](#2)
- [Project point on plane correction](#3)
- [Revit export geometry to AutoCAD via XML](#4)
- [RevitExportGeometryToAutocad](#4.1)
- [Description](#4.2)
- [Versions](#4.3)
- [Using](#4.4)
- [Example](#4.5)
#### Snoop Stable Representation of References
Alexander's raised
the [issue #40](https://github.com/jeremytammik/RevitLookup/issues/40) and subsequently
submitted [pull request #41](https://github.com/jeremytammik/RevitLookup/pull/41) to
display the result of the `ConvertToStableRepresentation` method when snooping `Reference` objects.
I integrated his improvements
in [RevitLookup](https://github.com/jeremytammik/RevitLookup)
[release 2018.0.0.7](https://github.com/jeremytammik/RevitLookup/releases/tag/2018.0.0.7).
Here is the result of snooping a reference from a dimension between two walls:
![Dimension reference stable representation](img/dimension_reference_stable_rep.png)
Many thanks to Alexander for his efficient enhancement!
Take a look at
the [diff from the previous version](https://github.com/jeremytammik/RevitLookup/compare/2018.0.0.6...2018.0.0.7) to
see how elegantly this was achieved.
#### Project Point on Plane Correction
Alexander also raised some other interesting issues in in the past in comments on
the [wall graph](http://thebuildingcoder.typepad.com/blog/2008/12/wall-graph.html#comment-3490286732),
[point in polygon algorithm](http://thebuildingcoder.typepad.com/blog/2010/12/point-in-polygon-containment-algorithm.html#comment-3504414240),
[wall elevation profile](http://thebuildingcoder.typepad.com/blog/2015/01/getting-the-wall-elevation-profile.html#comment-3759178237) and,
most recently and significantly,
on [projecting](http://thebuildingcoder.typepad.com/blog/2014/09/planes-projections-and-picking-points.html#comment-3765799540)
[a point](http://thebuildingcoder.typepad.com/blog/2014/09/planes-projections-and-picking-points.html#comment-3779858513)
[onto a plane](http://thebuildingcoder.typepad.com/blog/2014/09/planes-projections-and-picking-points.html#comment-3779960537),
uncovering an error in The Building Coder samples `ProjectOnto` method that projects a given 3D `XYZ` point onto a plane.
I originally presented this method in the discussion
on [planes, projections and picking points](http://thebuildingcoder.typepad.com/blog/2014/09/planes-projections-and-picking-points.html):
[projecting a 3D point onto a plane](http://thebuildingcoder.typepad.com/blog/2014/09/planes-projections-and-picking-points.html#12).
Swapping the sign seems to have fixed it, as proved
by [Alexander's ProjectPointOnPlanetest sample add-in](https://github.com/Pekshev/ProjectPointOnPlanetest):
```csharp
///
/// Project given 3D XYZ point onto plane.
/// summary>
public static XYZ ProjectOnto(
this Plane plane,
XYZ p )
{
double d = plane.SignedDistanceTo( p );
//XYZ q = p + d \* plane.Normal; // wrong according to Ruslan Hanza and Alexander Pekshev in their comments http://thebuildingcoder.typepad.com/blog/2014/09/planes-projections-and-picking-points.html#comment-3765750464
XYZ q = p - d \* plane.Normal;
Debug.Assert(
Util.IsZero( plane.SignedDistanceTo( q ) ),
"expected point on plane to have zero distance to plane" );
return q;
}
```
#### Revit Export Geometry to AutoCAD via XML
Browsing Alexander's other GitHub repositories, one that particularly caught my eye is his
[RevitExportGeometryToAutocad add-in](https://github.com/Pekshev/RevitExportGeometryToAutocad),
documented in Russian.
I read the Google-translated English description and think this sounds as if it might be very useful to others as well:
#### RevitExportGeometryToAutocad
Auxiliary libraries for rendering geometry from Revit to AutoCAD in the form of simple objects (a segment, an arc, a point) by exporting to XML.
#### Description
Libraries are useful in the development of plug-ins related to geometry, for convenient visual perception of the results.
In my opinion, viewing the result in AutoCAD is much more convenient.
This project provides two libraries (one for Revit, the second for AutoCAD) and a demo project for Revit.
#### Versions
The project for AutoCAD is built using libraries from AutoCAD 2013. It will work with all subsequent versions of AutoCAD.
The Revit project is built using Revit 2015 libraries. It should work with subsequent versions as well (tested for 2015-2018).
#### Using
The solution also contains a demo project for Revit.
Description of use for the example of this project:
\*\*In Revit\*\*
Connect to the project a link to the `RevitGeometryExporter.dll` library.
Before using export methods, you need to specify the folder to export xml
```csharp
// setup export folder
ExportGeometryToXml.FolderName = @ "C:\Temp";
```
By default, the library has the path \*C:\Temp\RevitExportXml\*.
In the absence of a directory, it will be created.
Call one or more methods for exporting geometry, for example:
```csharp
List wallsToExport = new List();
foreach( Reference reference in selectionResult )
{
Wall wall = (Wall) doc.GetElement( reference );
wallsToExport.Add( wall );
}
ExportGeometryToXml.ExportWallsByFaces( wallsToExport, "walls" );
```
Or
```csharp
List familyInstances = new List();
foreach( Reference reference in selectionResult )
{
Element el = doc.GetElement( reference );
if( el is FamilyInstance familyInstance)
familyInstances.Add( familyInstance );
}
ExportGeometryToXml.ExportFamilyInstancesByFaces( familyInstances, "families", false );
```
\*\*In AutoCAD\*\*
Use the NETLOAD command to load the `CadDrawGeometry.dll` library.
Use one of the two available commands:
- DrawFromOneXml – Draw geometry from one specified XML file
- DrawXmlFromFolder – Draw the geometry from the specified folder in which the XML files reside
#### Example
Elements in Revit:
![Elements in Revit](img/ap_rvt_to_xml_to_acad_1.png)
The result of exporting and rendering geometry in AutoCAD:
![Export result rendered in AutoCAD](img/ap_rvt_to_xml_to_acad_2.png)
Many thanks again to Alexander for sharing this!