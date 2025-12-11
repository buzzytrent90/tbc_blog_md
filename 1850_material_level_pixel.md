---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.888606'
original_url: https://thebuildingcoder.typepad.com/blog/1850_material_level_pixel.html
post_number: '1850'
reading_time_minutes: 11
series: materials
slug: material_level_pixel
source_file: 1850_material_level_pixel.md
tags:
- csharp
- elements
- family
- filtering
- levels
- parameters
- references
- revit-api
- rooms
- schedules
- sheets
- transactions
- views
- windows
- materials
title: Material Level Pixel
word_count: 2241
---

### Creating Material Texture and Retaining Pixels
I have been quiet now for a while in shock and grieving about violence and racism in the world.
Meanwhile, a bunch of interesting discussions on creating material with texture, modifying element level, cutting off image pixels and other things:
- [Creating a material with texture in Revit and Forge](#2)
- [Export image cutting off pixels](#3)
- [Change level of existing element](#4)
- [Physics is cool](#5)
- [Forge job openings](#6)
#### Creating a Material with Texture in Revit and Forge
This topic has been very much en vogue lately.
It came up again in the context of Forge in the StackOverflow question
on [creating a material with texture in Autodesk Revit Forge Design Automation](https://stackoverflow.com/questions/62297851/creating-a-material-with-texture-in-autodesk-revit-forge-design-automation),
where [maleficca](https://stackoverflow.com/users/12469767/maleficca) very kindly shares a complete solution for both environments:
\*\*Question:\*\* I'm currently working on some Revit API code which is running in the \*\*Autodesk Forge Design Automation cloud solution\*\*.
Basically, I'm trying to \*\*create a material and attach a texture to it\*\* via the following code:
```csharp
private void AddTexturePath(
AssetProperty asset,
string texturePath )
{
Asset connectedAsset = null;
if( asset.NumberOfConnectedProperties == 0 )
asset.AddConnectedAsset( "UnifiedBitmapSchema" );
connectedAsset = (Asset) asset.GetConnectedProperty( 0 );
AssetPropertyString path
= (AssetPropertyString) connectedAsset.FindByName(
UnifiedBitmap.UnifiedbitmapBitmap );
if( !path.IsValidValue( texturePath ) )
{
File.Create( "texture.png" );
texturePath = Path.GetFullPath( "texture.png" );
}
path.Value = texturePath;
}
```
This is actually working well, as the value for the texture path:
```csharp
path.Value = texturePath;
```
Needs to be a reference to an existing file. I do not have this file on the cloud instance of Forge, because the path to the texture name is specified by the user when he sends the request for the Workitem.
The problem is that this sets the texture path for the material as something like this:
```csharp
T:\Aces\Jobs\texture.png
```
Which is basically the working folder for the Workitem instance. This path is useless, because a material with texture path like this needs to be manually re-linked in Revit.
The perfect outcome for me would be if I could somehow map the material texture path to some user-friendly directory like `C:\Textures\texture.png` and it seems that the Forge instance has a `C:\` drive present (being probably a Windows instance of some sorts), but my code runs on low privileges, so it cannot create any kind of directories or files outside the working directory.
Does somebody have any idea how this could be resolved?
Any help would be greatly appreciated!
\*\*Answer:\*\* Congratulations on getting to this point.
Would you like to share the code you use to create the material and attach the texture for the Revit API add-in developer community to enjoy, either here or in a new thread in the Revit API discussion forum?
People keep asking for such samples... Thank you!
\*\*Response:\*\* Here is my own answer and code sample:
After a whole day of research, I pretty much arrived at a satisfying solution. Just for clarity – I am going to reference to \*\*Autodesk Forge Design Automation API for Revit\*\*, simply as \*\*"Forge"\*\*.
Basically, the code provided above is correct.
I did not find any possible way to create a file on Forge instance, in a directory different than the Workitem working directory which is:
```csharp
T:\Aces\Jobs\texture.png
```
Interestingly, there is a `C:\` drive on the Forge instance, which contains Windows, Revit and .NET Framework installations (as Forge instance is basically some sort of Windows instance with Revit installed). It is possible to enumerate a lot of these directories, but none of the ones I've tried (and I've tried a lot – mostly the most obvious, public access Windows directories like `C:\Users\Public`, `C:\Program Files`, etc.) allow for creation of directories or files. This corresponds to what is stated in "Restrictions" area of the Forge documentation:
> Your application is run with low privileges, and will not be able to freely interact with the Windows OS:
> - Write access is typically restricted to the job’s working folder.
> - Registry access is mostly restricted, writing to the registry should be avoided.
> - Any sub-process will also be executed with low privileges.
So, after trying to save the "dummy" texture file somewhere on the Forge `C:\` drive, I've found another solution – \*\*the texture path for your texture actually does not matter.\*\*
This is because Revit offers an alternative for re-linking your textures.
If you fire up Revit, you can go to File > Options > Rendering, and under "Additional render appearance paths" field, you can specify the directories on your local machine, that Revit can use to look for missing textures.
With these, you can do the following operations in order to have full control on creating materials on Forge:
1. Send Workitem to Forge, create the materials.
2. Create a dummy texture in working directory, with the correct file name.
3. Attach the dummy texture file to the material.
4. Output the resulting file (.rvt or .rfa, depending on what you're creating on Forge).
5. Place all textures into one folder (or multiple, this doesn't matter that much).
6. Add the directories with the textures to the Additional render appearance paths.
7. Revit will successfully re-link all the textures to new paths.
I hope someone will find this useful!
Additionally, as per Jeremy's request, I post a code sample for creating material with texture and modifying different Appearance properties in Revit by using Revit API (in C#):
```csharp
private void SetAppearanceParameters(
Document project,
Material mat,
MaterialData data )
{
using( Transaction setParameters = new Transaction(
project, "Set material parameters" ) )
{
setParameters.Start();
AppearanceAssetElement genericAsset
= new FilteredElementCollector( project )
.OfClass( typeof( AppearanceAssetElement ) )
.ToElements()
.Cast().Where( i
=> i.Name.Contains( "Generic" ) )
.FirstOrDefault();
AppearanceAssetElement newAsset
= genericAsset.Duplicate( data.Name );
mat.AppearanceAssetId = newAsset.Id;
using( AppearanceAssetEditScope editAsset
= new AppearanceAssetEditScope( project ) )
{
Asset editableAsset = editAsset.Start( newAsset.Id );
AssetProperty assetProperty
= editableAsset[ "generic_diffuse" ];
SetColor( editableAsset, data.MaterialAppearance.Color );
SetGlossiness( editableAsset, data.MaterialAppearance.Gloss );
SetReflectivity( editableAsset, data.MaterialAppearance.Reflectivity );
SetTransparency( editableAsset, data.MaterialAppearance.Transparency );
if( data.MaterialAppearance.Texture != null
&& data.MaterialAppearance.Texture.Length != 0 )
{
AddTexturePath( assetProperty,
$@"C:\{data.MaterialIdentity.Manufacturer}\textures\{data.MaterialAppearance.Texture}" );
}
editAsset.Commit( true );
}
setParameters.Commit();
}
}
private void SetTransparency(
Asset editableAsset,
int transparency )
{
AssetPropertyDouble genericTransparency
= editableAsset[ "generic_transparency" ]
as AssetPropertyDouble;
genericTransparency.Value = Convert.ToDouble(
transparency );
}
private void SetReflectivity(
Asset editableAsset,
int reflectivity )
{
AssetPropertyDouble genericReflectivityZero
= (AssetPropertyDouble) editableAsset[
"generic_reflectivity_at_0deg" ];
genericReflectivityZero.Value = Convert.ToDouble(
reflectivity ) / 100;
AssetPropertyDouble genericReflectivityAngle
= (AssetPropertyDouble) editableAsset[
"generic_reflectivity_at_90deg" ];
genericReflectivityAngle.Value = Convert.ToDouble(
reflectivity ) / 100;
}
private void SetGlossiness(
Asset editableAsset,
int gloss )
{
AssetPropertyDouble glossProperty
= (AssetPropertyDouble) editableAsset[
"generic_glossiness" ];
glossProperty.Value = Convert.ToDouble(
gloss ) / 100;
}
private void SetColor(
Asset editableAsset,
int[] color )
{
AssetPropertyDoubleArray4d genericDiffuseColor
= (AssetPropertyDoubleArray4d) editableAsset[
"generic_diffuse" ];
Color newColor = new Color( (byte) color[ 0 ],
(byte) color[ 1 ], (byte) color[ 2 ] );
genericDiffuseColor.SetValueAsColor( newColor );
}
private void AddTexturePath(
AssetProperty asset,
string texturePath )
{
Asset connectedAsset = null;
if( asset.NumberOfConnectedProperties == 0 )
asset.AddConnectedAsset( "UnifiedBitmapSchema" );
connectedAsset = (Asset) asset.GetConnectedProperty( 0 );
AssetProperty prop = connectedAsset.FindByName(
UnifiedBitmap.UnifiedbitmapBitmap );
AssetPropertyString path
= (AssetPropertyString) connectedAsset.FindByName(
UnifiedBitmap.UnifiedbitmapBitmap );
string fileName = Path.GetFileName( texturePath );
File.Create( fileName );
texturePath = Path.GetFullPath( fileName );
path.Value = texturePath;
}
```
Hopefully it will come in handy to someone in the future!
Also, a huge thanks for The Building Coder; it has saved me a lot of hassle in my work with Revit API and enabled to create a lot of cool stuff!
A great thanks back to maleficca for kindly sharing the two solutions, both for Forge and the Revit desktop API!
#### Export Image Cutting off Pixels
This query just came up again in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [export image cutting off edge pixels](https://forums.autodesk.com/t5/revit-api-forum/export-image-cutting-off-edge-pixels/m-p/9578304):
\*\*Question:\*\* I'm trying to export family type images from a family, but I'm getting the edge pixels cut off in my exported images.
Here is the original image:
![Original image](img/image_export_cutting_pixels_original.png "Original image")
This is what comes out:
![Exported image](img/image_export_cutting_pixels_exported.png "Exported image")
This is what I currently have for my export image options
```csharp
img = ImageExportOptions()
img.ZoomType = ZoomFitType.FitToPage
img.PixelSize = size
img.ImageResolution = ImageResolution.DPI_150
img.FitDirection = FitDirectionType.Vertical
img.ExportRange = ExportRange.SetOfViews
img.SetViewsAndSheets( viewIds )
img.HLRandWFViewsFileType = ImageFileType.PNG
img.FilePath = filepath
img.ShadowViewsFileType = ImageFileType.PNG
```
A solution was provided by [alexpaduroiu](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/7761409) in a previous conversation
on why [export image is cutting a few pixels from image corners](https://forums.autodesk.com/t5/revit-api-forum/export-image-is-cutting-few-pixels-from-image-corners/m-p/9346019):
\*\*Question:\*\* I have a small problem regarding `Decoument.ExportImage(ImageExportOptions options)`.
I am trying to export a set of drafting views, but somehow the generated images cut the views edges.
A sample image:
![Incomplete image](img/image_export_cutting_pixels_incomplete.png "Incomplete image")
This image has the bottom part not visible at all:
![Incomplete image](img/image_export_cutting_pixels_incomplete2.png "Incomplete image")
The cut is very small but very frustrating because I can't make it a whole or even offset the element to fit the images.
Image Export Options used:
```csharp
ImageExportOptions imgExportOpts
= new ImageExportOptions()
{
ZoomType = ZoomFitType.FitToPage,
PixelSize = 500,
FilePath = rbrImagesDirectory + @"\",
FitDirection = FitDirectionType.Vertical,
HLRandWFViewsFileType = ImageFileType.PNG,
ShadowViewsFileType = ImageFileType.PNG,
ImageResolution = ImageResolution.DPI_72,
ShouldCreateWebSite = false,
};
imgExportOpts.ExportRange = ExportRange.SetOfViews;
```
I have tried to modify `FitDirectionType.Horizontal` and this makes it worse than it is now by cutting the bottom portion even more; in that case, for the first image, the bottom part of the bar is not visible at all.
The image doesn't have such a big cut in the edges but it will be nice to have some spaces there or at least to see the parts :-)
Is there any way to zoom out the element or move it in order to be arranged better in image?
\*\*Answer:\*\* Well, I have solved the problem!
The problem was with the drafting view I was trying to export. After creating a group of details in the drafting view, somehow the outline of the view wasn't big enough to include my details. don't know for sure why, but what I have done to resolve the problem was the following:
```csharp
drafting.CropBoxActive = true;
drafting.CropBoxVisible = true;
private static void ExtendViewCrop(
View drafting,
Group detail )
{
BoundingBoxXYZ crop = (drafting != null
? drafting.CropBox
: null);
if( crop == null || crop.Max == null || crop.Min == null
|| crop.Transform == null || detail == null )
{
return;
}
BoundingBoxXYZ detailBox = detail.get_BoundingBox( drafting );
BoundingBoxXYZ extendedCrop = new BoundingBoxXYZ();
extendedCrop.Transform = crop.Transform;
if( detailBox == null || detailBox.Max == null
|| detailBox.Min == null )
{
return;
}
extendedCrop.Max = detailBox.Max
+ (extendedCrop.Transform.BasisX
+ extendedCrop.Transform.BasisY / 2) \* 0.03;
extendedCrop.Min = detailBox.Min
+ (-extendedCrop.Transform.BasisX
+ -extendedCrop.Transform.BasisY / 2) \* 0.03;
drafting.CropBox = extendedCrop;
}
```
So basically, extending the view crops max and min on their direction with a small value so the detail will fit in my drafting.
I am sure that there are lots of other better options doing this, but for now it did the trick.
Of course I am open to more solutions :-)
Many thanks to alexpaduroiu for this clear solution.
#### Change Level of Existing Element
Angelo Mastroberardino brought up this question in
his [comment](https://thebuildingcoder.typepad.com/blog/2014/03/creating-a-sloped-floor.html#comment-4952460602)
on [creating a sloped floor](https://thebuildingcoder.typepad.com/blog/2014/03/creating-a-sloped-floor.html):
\*\*Question:\*\* Is it possible to re-set the reference Level of a Floor, once it is created ?
\*\*Answer:\*\* In general, the `Level` property is read-only and thus cannot be set after an element has been created.
It is specified during creation only and cannot be modified later.
Here are some posts on levels, both general level handling issues and workarounds to set the level property on certain specific element types:
- [Retrieve levels sorted by elevation](https://thebuildingcoder.typepad.com/blog/2014/11/webgl-goes-mobile-and-sorted-level-retrieval.html#3)
- [Family instances lacking or with invalid level property](https://thebuildingcoder.typepad.com/blog/2011/01/family-instance-missing-level-property.html)
- [Changing the level of a floor](https://thebuildingcoder.typepad.com/blog/2019/04/set-floor-level-and-use-ipc-for-disentanglement.html#3)
- [Calculate a level for an element lacking the property](https://thebuildingcoder.typepad.com/blog/2019/03/assigning-a-level-to-an-element-missing-it.html)
- A solution to [change the level of an existing room](https://thebuildingcoder.typepad.com/blog/2020/03/panel-schedule-slots-and-change-room-level.html)
#### Physics is Cool
A very nice and surprising [physics experiment to try out at home](https://www.reddit.com/r/BeAmazed/comments/gxrq8p/physics_is_cool):
[![Physics is cool](img/cool_physics.png "Physics is cool")](https://www.reddit.com/r/BeAmazed/comments/gxrq8p/physics_is_cool)
#### Forge Job Openings
Are you a critical thinker, problem solver, story teller, goal oriented, smart, curios, empathic, with experience in a cloud environment such as AWS?
If so, would you like to consider applying for a job in the Forge team?
- [20WD39627 – Senior Vendor Manager – San Francisco](https://rolp.co/SS7ji)
- [20WD38934 – Localization Software Engineer – Singapore](https://rolp.co/4Obwi)
- [20WD37407 – Senior Product Manager, Data – Montreal](https://rolp.co/Q6cUi)
- [20WD40315 – Senior Data Engineer/Architect – Novi ](https://rolp.co/pVKii)
Good luck applying for one of these or the many other opportunities that you can find all over the world in
the [Autodesk career site](https://www.autodesk.com/careers)!