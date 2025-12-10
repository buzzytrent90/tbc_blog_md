---
post_number: "1522"
title: "Revitserverapilib"
slug: "revitserverapilib"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'revit-api', 'sheets']
source_file: "1522_revitserverapilib.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1522_revitserverapilib.html"
---

### Revit Server API Lib, Truss Members and Layers
Today, we proudly present:
- [NuGet Revit Server REST API Library](#2)
- [RevitLookup Updates](#3)
- [Truss Members and FamilyInstance Sub-Components](#4)
- [GeometryObject Layer Name](#5)
#### NuGet Revit Server REST API Library
Eric Anastas posted
a [comment](http://thebuildingcoder.typepad.com/blog/2016/12/nuget-revit-api-package.html#comment-3133102375) on
the discussion of Andrey Bushman's [NuGet Revit API package](http://thebuildingcoder.typepad.com/blog/2016/12/nuget-revit-api-package.html):
> I created another Revit related Nuget package you and your readers may be interested in.
It's a .NET library that wraps the Revit Server REST API:
- [RevitServerAPILib package on NuGet ](https://www.nuget.org/packages/RevitServerAPILib)
- [RevitServerAPILib source on BitBucket](https://bitbucket.org/somddg/revitserverapilib)
Many thanks to Eric for implementing and sharing this!
!
#### RevitLookup Updates
I integrated another couple of pull requests into RevitLookup.
Today, I added [Einar Raknes](https://github.com/eibre)' simple but significant one-liner
to [display the `UnitType` or a parameter `Definition` class instance](https://github.com/jeremytammik/RevitLookup/compare/2017.0.0.11...2017.0.0.12):
```csharp
data.Add( new Snoop.Data.String( "Unit type",
paramDef.UnitType.ToString() ) );
```
Thanks to Einar for spotting this and creating the [pull request #21](https://github.com/jeremytammik/RevitLookup/pull/21) for it!
Here it is with a little bit more context:
```csharp
private void
Stream( ArrayList data, Definition paramDef )
{
data.Add( new Snoop.Data.ClassSeparator( typeof( Definition ) ) );
data.Add( new Snoop.Data.String( "Name", paramDef.Name ) );
data.Add( new Snoop.Data.String( "Parameter type", paramDef.ParameterType.ToString() ) );
data.Add( new Snoop.Data.String( "Parameter group", paramDef.ParameterGroup.ToString() ) );
data.Add( new Snoop.Data.String( "Unit type", paramDef.UnitType.ToString() ) );
ExternalDefinition extDef = paramDef as ExternalDefinition;
if( extDef != null )
{
Stream( data, extDef );
return;
}
InternalDefinition intrnalDef = paramDef as InternalDefinition;
if( intrnalDef != null )
{
Stream( data, intrnalDef );
return;
}
}
```
Check out the newest release in the [RevitLookup GitHub repository](https://github.com/jeremytammik/RevitLookup).
I am looking forward to your pull requests to add further enhancements that are important for you.
#### Truss Members and FamilyInstance Sub-Components
I have been pretty active lately in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
and so have Matt Taylor and Frank 'Fair59', who provided many important answers that I was not aware of.
I'll pick up two of Franks nice succinct answers here today.
The first is on retrieving the [components of a truss](http://forums.autodesk.com/t5/revit-api-forum/component-of-a-truss/m-p/6845784):
\*\*Question:\*\* I would like to get the components of a truss.
Using the API, I tried via the `GroupId` property, but it doesn't work.
And in general, the components of a family.
\*\*Answer:\*\* Components of a truss:
```csharp
Autodesk.Revit.DB.Structure.Truss _truss;
List _members = _truss.Members.ToList();
```
In general, for user created families:
```csharp
FamilyInstance _instance;
List _members = _instance.GetSubComponentIds()
.ToList();
```
#### GeometryObject Layer Name
The second nice succinct answer by Frank 'Fair59' is on
the [`GeometryObject` Layer Name](http://forums.autodesk.com/t5/revit-api-forum/geometryobject-layer-name/m-p/6835165):
\*\*Question:\*\* I can loop the objects of a linked CAD file like this:
```csharp
foreach( GeometryObject geometryObj in
dwg.get_Geometry( new Options() ) )
{
}
```
How can I access the layer name of the object?
\*\*Answer:\*\* The information is contained in the `GraphicalStyle` element:
```csharp
GraphicsStyle gStyle = document.GetElement(
geometryObj.GraphicsStyleId ) as GraphicsStyle;
```
The layer name is provided by `gStyle.GraphicsStyleCategory.Name`.