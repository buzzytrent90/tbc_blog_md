---
post_number: "1596"
title: "Texture Path"
slug: "texture_path"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'geometry', 'levels', 'references', 'revit-api', 'rooms', 'sheets']
source_file: "1596_texture_path.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1596_texture_path.html"
---

### Material Texture Path
Here is an official answer from the Revit development team on the long-standing and recurrent issue on retrieving the path to a specific material texture bitmap file:
\*\*Question:\*\* I am working on an exporter plugin for Revit that exports all geometry from selected objects using the `CustomExporter` framework.
When extracting object materials, I can successfully get most of the information, but I can't seem to find the path to the material texture.
I already asked this in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [extract object texture information using API](https://forums.autodesk.com/t5/revit-api-forum/extract-object-texture-information-using-api/m-p/7406055) without
receiving any useful answer.
Is there a way to do this or is it a limitation in the API?
I already tried and failed to achieve this in Navisworks. Hopefully it is more easily done in Revit.
In Navisworks, when we add models from other CAD systems, the right textures are displayed, so why don't we have access to that same info when we try to export data?
We will soon have no choice but to abandon this idea and move on to something else that can support this type of workflow, because our customers are all the time asking for the textures also, and we are currently forced to do huge work by hand to provide those.
\*\*Answer:\*\* The development team responds:
The Building Coder article
on [texture bitmap and `UV` coordinates](http://thebuildingcoder.typepad.com/blog/2013/07/texture-bitmap-and-uv-coordinates.html) part 1
on [obtaining texture `UV` coordinates](http://thebuildingcoder.typepad.com/blog/2013/07/texture-bitmap-and-uv-coordinates.html#2) covers the `CustomExporter`, which is still the best way to extract information on materials and their mapping onto faces.
If you are trying to export an actual representation of an actual Revit model, you really should start with `CustomExporter`, regardless of the target format.
Part 2
on [texture bitmap access](http://thebuildingcoder.typepad.com/blog/2013/07/texture-bitmap-and-uv-coordinates.html#2) says 'the texture `UV` coordinates are available, and the bitmap is not.'
I now realize that we have a problem here. We've long been able to read the properties of assets, including the associated bitmaps. The Revit API provides access to a relative path for any bitmap asset in any material. Here is a code sample demonstrating this:
```csharp
string[] targetMaterialNames = {
// A standard Revit material, with
// textures in standard paths.
"Brick, Common",
// A material with a single image from
// another non-material library path
"Local Path Material"
};
void FindTextureBitmapPaths( Document doc )
{
// Find materials
FilteredElementCollector fec
= new FilteredElementCollector( doc );
fec.OfClass( typeof( Material ) );
IEnumerable targetMaterials
= fec.Cast().Where( mtl =>
targetMaterialNames.Contains( mtl.Name ) );
foreach( Material material in targetMaterials )
{
// Get appearance asset for read
ElementId appearanceAssetId = material
.AppearanceAssetId;
AppearanceAssetElement appearanceAssetElem
= doc.GetElement( appearanceAssetId )
as AppearanceAssetElement;
Asset asset = appearanceAssetElem
.GetRenderingAsset();
// Walk through all first level assets to find
// connected Bitmap properties. Note: it is
// possible to have multilevel connected
// properties with Bitmaps in the leaf nodes.
// So this would need to be recursive.
int size = asset.Size;
for( int assetIdx = 0; assetIdx < size; assetIdx++ )
{
AssetProperty aProperty = asset[assetIdx];
if( aProperty.NumberOfConnectedProperties < 1 )
continue;
// Find first connected property.
// Should work for all current (2018) schemas.
// Safer code would loop through all connected
// properties based on the number provided.
Asset connectedAsset = aProperty
.GetConnectedProperty( 0 ) as Asset;
// We are only checking for bitmap connected assets.
if( connectedAsset.Name == "UnifiedBitmapSchema" )
{
// This line is 2018.1 & up because of the
// property reference to UnifiedBitmap
// .UnifiedbitmapBitmap. In earlier versions,
// you can still reference the string name
// instead: "unifiedbitmap_Bitmap"
AssetPropertyString path = connectedAsset[
UnifiedBitmap.UnifiedbitmapBitmap]
as AssetPropertyString;
// This will be a relative path to the
// built -in materials folder, addiitonal
// render appearance folder, or an
// absolute path.
TaskDialog.Show( "Connected bitmap",
String.Format( "{0} from {2}: {1}",
aProperty.Name, path.Value,
connectedAsset.LibraryName ) );
}
}
}
}
```
This code provides the bitmap path.
There is still one problem, though: it is often a relative path, either to the default Materials library location, e.g. \*C:\Program Files (x86)\Common Files\Autodesk Shared\Materials\Textures\*, or a user-specific \*Additional render appearance path\*. It could also be an absolute path if it's not in the materials folder or the additional paths.
I think that, to actually make this work, you will need some additional logic in your source code. If it's an absolute path, just use it. If it's a relative path, check if the file exists relative to the default material library installation. If not, read the `Revit.ini` file to find the other root path(s) and check each location in turn.
![Material texture](img/rh_texture_01.png)
For the sake of completeness, here are pointers to some related issues:
- [Listing material asset textures and sub-textures](http://thebuildingcoder.typepad.com/blog/2016/10/list-material-asset-texture-and-forge-webinar-recordings.html#3)
- [Retrieve and map texture `UV` coordinates exporting geometry and material](http://thebuildingcoder.typepad.com/blog/2017/03/events-uv-coordinates-and-rooms-on-level.html#5)
- [The new Visual Materials API in Revit 2018.1](http://thebuildingcoder.typepad.com/blog/2017/08/revit-20181-and-the-visual-materials-api.html)