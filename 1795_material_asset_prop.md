---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:16.762373'
original_url: https://thebuildingcoder.typepad.com/blog/1795_material_asset_prop.html
post_number: '1795'
reading_time_minutes: 5
series: materials
slug: material_asset_prop
source_file: 1795_material_asset_prop.md
tags:
- elements
- filtering
- parameters
- revit-api
- sheets
- views
- materials
title: Material Asset Prop
word_count: 1034
---

### Material, Physical and Thermal Assets
Today, we concentrate fully on material, physical and thermal assets:
- [Issues accessing and creating material assets](#2)
- [Access to all material asset properties](#3)
- [No access to material keywords](#4)
- [Access to environment and render settings](#5)
- [Determine full path to texture bitmap](#6)
- [Addendum – Modifying Material Assets](#7)
#### Issues Accessing and Creating Material Assets
Quite a number of issues were discussed in the past few months involving issues accessing and creating material assets.
As far as I know, they can all be solved, [cf. below](#3).
Sometimes, though, the access is slightly convoluted and confusing.
Here is an overview over some recent issues:
- [Create new material and add texture from file path](https://stackoverflow.com/questions/58414284/revit-api-material-with-texture-from-filepath)
- [Create new material and add new physical and thermal assets with values](https://forums.autodesk.com/t5/revit-api-forum/create-new-material-and-add-new-physical-and-thermal-assets-with/m-p/7311468)
- [Setting material texture path in `EditScope`](https://thebuildingcoder.typepad.com/blog/2019/04/set-material-texture-path-in-editscope.html) – ([discussion forum thread](https://forums.autodesk.com/t5/revit-api-forum/changing-material-texture-path-with-editscope/m-p/8017578), case 14254618)
Older discussions are listed in the topic groups
on [material management and libraries](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.5)
and [texture bitmap and UV coordinate access](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.42).
#### Access to All Material Asset Properties
I am very glad to announce that my colleagues Liz Fortune and Balaji Arunachalam confirm that they can access each and every parameter of the materials, appearance assets, thermal assets, and physical assets (with one single exception, [cf. below](4)):
Balaji and I have been able to access each and every parameter of the material, appearance asset, thermal asset, and physical asset EXCEPT for the keywords parameter that lives on the Identity tab of the Material UI in Revit. We can connect to the keywords for all of the assets above but would like to also connect to the keywords that live on the material itself.
To access this data, we used code snippets shared on The Building Coder blog to implement the following:
```csharp
void GetElementMaterialInfo( Document doc )
{
FilteredElementCollector collector
= new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.OfClass( typeof( Material ) );
try
{
foreach( Material material in collector )
{
if( material.Name.Equals( "Air" ) )
{
AppearanceAssetElement appearanceElement
= doc.GetElement( material.AppearanceAssetId )
as AppearanceAssetElement;
Asset appearanceAsset = appearanceElement
.GetRenderingAsset();
List assetProperties
= new List();
PropertySetElement physicalPropSet
= doc.GetElement( material.StructuralAssetId )
as PropertySetElement;
PropertySetElement thermalPropSet
= doc.GetElement( material.ThermalAssetId )
as PropertySetElement;
ThermalAsset thermalAsset = thermalPropSet
.GetThermalAsset();
StructuralAsset physicalAsset = physicalPropSet
.GetStructuralAsset();
ICollection physicalParameters
= physicalPropSet.GetOrderedParameters();
ICollection thermalParameters
= thermalPropSet.GetOrderedParameters();
// Appearance Asset
for( int i = 0; i < appearanceAsset.Size; i++ )
{
AssetProperty property = appearanceAsset[ i ];
assetProperties.Add( property );
}
foreach( AssetProperty assetProp in assetProperties )
{
Type type = assetProp.GetType();
object assetPropValue = null;
var prop = type.GetProperty( "Value" );
if( prop != null
&& prop.GetIndexParameters().Length == 0 )
{
assetPropValue = prop.GetValue( assetProp );
}
}
// Physical (Structural) Asset
foreach( Parameter p in physicalParameters )
{
// Work with parameters here
// The only parameter not in the orderedParameters
// that is needed is the Asset name, which you
// can get by 'physicalAsset.Name'.
}
// Thermal Asset
foreach( Parameter p in thermalParameters )
{
// Work with parameters here
// The only parameter not in the orderedParameters
// that is needed is the Asset name, shich you
// can get by 'thermalAsset.Name'.
}
}
}
}
catch( Exception e )
{
Console.WriteLine( e.ToString() );
}
}
```
This code will get you everything for the thermal and structural assets and almost everything for the appearance asset.
[Boris Shafiro's PDF](zip/sd124625_visual_appearance_of_materials_api_boris_shafiro_handout.pdf) from his AU presentation
on [modifying material visual appearance](https://thebuildingcoder.typepad.com/blog/2017/11/modifying-material-visual-appearance.html)
was crucial in figuring out how to touch all the aspects of the appearance asset!
Here is an image I put together [showing mappings for the appearance asset](zip/assets_air_appearance_properties.png):
![Air appearance properties](img/assets_air_appearance_properties.png)
The spreadsheet in this image was created from output of my code into a CSV. Hope this helps.
Many thanks to Liz and Balaji for their research and sharing the results!
#### No Access to Material Keywords
As mentioned above, one single property is apparently still not accessible: the keywords parameter that lives on the Identity tab of the Material UI in Revit.
Apparently, the internal property `Material.getKeywords` is not exposed to the official Revit API, and there is no built-in parameter for it either.
#### Access to Environment and Render Settings
We also searched for the Environment and Render Settings values. They are common for all the Assets:
![Environment assets](img/assets_environment.png)

Environment assets

![Render settings](img/assets_render_settings.png)

Render settings

They are common for all assets, so they look like the other properties or parameters, but are in fact part of the UI config and saved into `MaterialUIConfig.xml`.
#### Determine Full Path to Texture Bitmap
Finally, to round off this collection, here is a result from
a [comment](https://thebuildingcoder.typepad.com/blog/2016/10/list-material-asset-texture-and-forge-webinar-recordings.html#comment-4442686784)
on [material asset textures](https://thebuildingcoder.typepad.com/blog/2016/10/list-material-asset-texture-and-forge-webinar-recordings.html):
\*\*Question:\*\* Great tip, but how can we create the full path to the `unifiedbitmap_Bitmap` texture?
I tried using the `Application` `GetLibraryPaths` method, but the material library is not listed in the dictionary. :(
\*\*Answer:\*\* If all else fails, you can try to use the operating system functionality to search globally for the given filename.
You should cache the folders in which you find it.
If the system is sensibly set up, you will only need to search for it globally once, or a very few times, since there should not be many different locations in which these files are stored.
\*\*Response:\*\* Thank you for this tip, especially for the cache recommendation.
Works like a charm. :)
#### Addendum – Modifying Material Assets
\*\*Question:\*\* Can you also confirm that you can create new materials and successfully populate all these properties as well?
\*\*Answer:\*\* I have successfully modified the values for the parameters.
I attempted a handful.
The trick I found was that I needed to identify the storage type of each so I could set the parameter accordingly.