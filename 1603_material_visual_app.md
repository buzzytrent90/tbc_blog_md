---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.6
content_type: qa
optimization_date: '2025-12-11T11:44:16.374540'
original_url: https://thebuildingcoder.typepad.com/blog/1603_material_visual_app.html
post_number: '1603'
reading_time_minutes: 5
series: materials
slug: material_visual_app
source_file: 1603_material_visual_app.md
tags:
- elements
- references
- revit-api
- sheets
- views
- walls
- materials
title: The Basics
word_count: 936
---

### Modifying Material Visual Appearance
Several queries concerning rendering issues were discussed recently and solved by the new Visual Materials API included in Revit 2018.1:
- [Revit 2018.1 and the Visual Materials API](http://thebuildingcoder.typepad.com/blog/2017/08/revit-20181-and-the-visual-materials-api.html)
- [Appearance Asset Editing SDK sample](http://thebuildingcoder.typepad.com/blog/2017/08/revit-20181-and-the-visual-materials-api.html#2)
Here comes another one, answered more completely by Boris Shafiro's Autodesk University class on this topic:
- AU class catalogue entry: [SD124625 – New API to Modify Visual Appearance of Materials in Revit](https://autodeskuniversity.smarteventscloud.com/connect/sessionDetail.ww?SESSION_ID=124625)
- Class handout and slides: [SD124625 – Visual Materials API](#3)
\*\*Question:\*\* How can I set the Material Render Appearance through the API in Revit 2018?
I can see there is the `Autodesk.Revit.DB.Visual.Asset` class, but how do I add to the list of `Autodesk.Revit.DB.Visual.AssetProperty` objects for a new material?
I noticed the forum thread
on how to [create or modify a rendering asset](https://forums.autodesk.com/t5/revit-api-forum/create-or-modify-a-rendering-asset/td-p/6244577) that
seems to indicate limitations in this area...
\*\*Answer:\*\* Unfortunately, this is not possible in Revit 2018.
The good news is that it ***is*** possible in Revit 2018.1 using the Visual Materials API.
Check out Boris Shafiro's class at AU to learn about it.
![Visual Materials API](img/sd124625_visual_materials_api.png)
#### SD124625 – Visual Materials API
The ability to use the Revit API to modify visual appearances of materials was among the top customer requests for years. This new API has been implemented in Revit 2018.1. This class presents the new Visual Materials API, coding workflows, and usage of multiple schemas for the visualization properties of materials in Revit software – by Boris Shafiro, Software Development Manager, Autodesk:
- [Handout document PDF](/a/doc/au/2017/doc2/sd124625_Visual_Appearance_of_Materials_API_Boris_Shafiro_handout.pdf)
- [Slide deck PDF](/a/doc/au/2017/doc2/sd124625_Visual_Appearance_of_Materials_API_Boris_Shafiro_slides.pdf)
For the sake of completeness and search engine findability, here is the pure text copied out of the slide deck:
- [Learning Objectives](#3.1)
- [The Basics](#4)
- [Materials API](#4.1)
- [Terminology](#4.2)
- [Material API building blocks](#4.3)
- [Visual Materials UI](#4.4)
- [New Editing Capabilities in Materials API](#5)
- [Edit Scope](#5.1)
- [New Writable Properties](#5.2)
- [New Methods](#5.3)
- [Coding Workflow to Edit a Color](#5.4)
- [Connected Assets](#5.5)
- [Coding Workflow to Edit a Connected Asset](#5.6)
- [Schemas and Property Names](#7)
- [Standard Material Schemas](#7.1)
- [Advanced Material Schemas](#7.2)
- [Common Schema](#7.3)
- [Schemas for Connected Assets](#7.4)
- [UnifiedBitmap](#7.5)
- [Property Names](#7.6)
- [Special Cases](#7.7)
- [SDK Sample](#8)
- [AppearanceAssetEditing](#8.1)
#### Learning Objectives
Learn how to
- Use new API to modify visual appearance of Materials in Revit
- Navigate coding workflow to edit appearance assets
- Use multiple schemas for regular and advanced materials in Revit
- Write a sample plug-in for basic modification of the visual appearance of Revit materials

### The Basics

#### Materials API
- Basic Element Info (name, tags)
- Appearance properties
- Shaded view graphics
- Thermal & energy-related properties
- Physical & structural properties
#### Terminology
- Revit Material – An element representing a material, made of a collection of property sets
- Asset – The class representing a package of properties
- Appearance Asset – Asset representing visual material properties
- Appearance Asset Element – An element that stores an appearance asset
- Asset Property – One particular property of an asset
#### Material API building blocks
- Namespace Revit.DB Namespace Revit.DB.Visual
- Material – AppearanceAssetId
- AppearanceAssetElement – GetRenderingAsset()
- Asset – AssetProperty 1 ... AssetProperty N – [“name_string” ] or FindByName(name)
- AssetProperty – GetSingleConnectedAsset()
#### Visual Materials UI

### New Editing Capabilities in Materials API

#### Edit Scope
- AppearanceAssetEditScope
- Start()
- Commit()
- Cancel()
- Contains one Asset (plus all connected Assets)
#### New Writable Properties
- AssetPropertyString.Value
- AssetPropertyBoolean.Value
- AssetPropertyInteger.Value
- AssetPropertyDouble.Value
- AssetPropertyFloat.Value
- AssetPropertyEnum.Value
- AssetPropertyDistance.Value (not always in feet)
#### New Methods
- AssetPropertyDoubleArray3d.SetValueAsXYZ()
- AssetPropertyDoubleArray4d.SetValueAsDoubles()
- AssetPropertyDoubleArray4d.SetValueAsColor()
- AssetPropertyList – add, insert, remove
#### Coding Workflow to Edit a Color
```csharp
using( AppearanceAssetEditScope editScope
= new AppearanceAssetEditScope( document ) )
{
Asset editableAsset = editScope.Start( assetElem.Id );
AssetPropertyDoubleArray4d genericDiffuseProperty
= editableAsset["generic_diffuse"]
as AssetPropertyDoubleArray4d;
genericDiffuseProperty.SetValueAsColor( color );
editScope.Commit( true );
}
```
#### Connected Assets
- AssetProperty.GetSingleConnectedAsset()
- AssetProperty.RemoveConnectedAsset()
- AssetProperty.AddConnectedAsset( String schemaId )
- AssetProperty.AddCopyAsConnectedAsset( Asset renderingAsset )
#### Coding Workflow to Edit a Connected Asset
```csharp
using( AppearanceAssetEditScope editScope
= new AppearanceAssetEditScope( document ) )
{
Asset editableAsset = editScope.Start( assetElem.Id );
AssetProperty bumpMapProperty = editableAsset["generic_bump_map"];
Asset connectedAsset = bumpMapProperty.GetSingleConnectedAsset();
if( connectedAsset != null )
{
AssetPropertyString bumpmapBitmapProperty
= connectedAsset["unifiedbitmap_Bitmap"]
as AssetPropertyString;
if( bumpmapBitmapProperty.IsValidValue( bumpmapImageFilepath ) )
bumpmapBitmapProperty.Value = bumpmapImageFilepath;
}
editScope.Commit( true );
}
```

### Schemas and Property Names

#### Standard Material Schemas
- Ceramic
- Concrete
- Generic
- Glazing
- Hardwood
- MasonryCMU
- Metal
- MetallicPaint
- Mirror
- PlasticVinyl
- SolidGlass
- Stone
- WallPaint
- Water
#### Advanced Material Schemas
- AdvancedLayered
- AdvancedMetal
- AdvancedOpaque
- AdvancedTransparent
- AdvancedWood
#### Common Schema
#### Schemas for Connected Assets
- BumpMap
- Checker
- Gradient
- Marble
- Noise
- Speckle
- Tile
- UnifiedBitmap
- Wave
- Wood
#### UnifiedBitmap
#### Property Names
```csharp
AssetPropertyDoubleArray4d genericDiffuseProperty
= editableAsset["generic_diffuse"]
as AssetPropertyDoubleArray4d;
```
Equivalent:
```csharp
AssetPropertyDoubleArray4d genericDiffuseProperty
= editableAsset[Generic.GenericDiffuse]
as AssetPropertyDoubleArray4d;
```
#### Special Cases
```csharp
AssetPropertyString path
= asset[UnifiedBitmap.UnifiedbitmapBitmap]
as AssetPropertyString;
```
- Path is relative if inside default Material Library or in Options/Rendering/Additional Render Appearance Paths;
- Path is absolute otherwise.
```csharp
AssetPropertyDoubleArray4d color
= asset[Generic.DiffuseColor]
as AssetPropertyDoubleArray4d;
```
- The Value of this AssetProperty is ignored if there is a connected Asset.
- AssetPropertyReference reference – Does not have a Value. Used only to have a connected Asset.

### SDK Sample

#### AppearanceAssetEditing
Edit appearance asset properties via a small control dialog – this sample demonstrates basic usage of the AppearanceAssetEditScope and AssetProperty classes to change the value of an asset property in a given material:
- Bring up a modeless dialog
- Select a Painted Face
- Get Appearance Asset
- Get Tint Color AssetProperty
- Increment red/green/blue