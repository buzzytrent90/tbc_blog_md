---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 9.0
content_type: qa
optimization_date: '2025-12-11T11:44:17.072550'
original_url: https://thebuildingcoder.typepad.com/blog/1932_symb_inst_ole_journal.html
post_number: '1932'
reading_time_minutes: 17
series: general
slug: symb_inst_ole_journal
source_file: 1932_symb_inst_ole_journal.md
tags:
- doors
- elements
- family
- filtering
- geometry
- parameters
- references
- revit-api
- rooms
- selection
- sheets
- transactions
- views
- windows
title: Symb Inst Ole Journal
word_count: 3342
---

### Symbol, Instance, Material, Data, Journal, Break
This is probably my last post of the year, so let's wrap up with an eclectic mix of topics to close with:
- [Symbol vs instance geometry clarification](#2)
- [Create new material with texture](#3)
- [Updates from Devteam, Richard, and Harm](#3.2)
- [RVT dashboard data access](#4)
- [Marking and retrieving a custom element](#5)
- [Advanced remote batch command processing](#6)
- [Midwinter break](#7)
#### Symbol vs Instance Geometry Clarification
Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas
clarifies some aspects of symbol versus instance geometry in an imported DWG file in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`GetInstanceGeometry` vs `GetSymbolGeometry`](https://forums.autodesk.com/t5/revit-api-forum/getinstancegeometry-vs-getsymbolgeometry/m-p/10819201):
\*\*Question:\*\* ... about the methods GeometryInstance.GetInstanceGeometry() and GeometryInstance.GetSymbolGeometry().
In my file, I've got only one imported DWG with only one line inside it.
I analysed it using the following code:
```csharp
var reference = _selection.PickObject(ObjectType.PointOnElement);
var element = _doc.GetElement(reference);
var options = new Options();
options.ComputeReferences = true;
options.View = _doc.ActiveView;
var geometryElement = element.get_Geometry(options);
var geometryInstance = geometryElement
.FirstOrDefault(x => x is GeometryInstance) as
GeometryInstance;
var instanceGeometry = geometryInstance?.GetInstanceGeometry();
var instanceCurve = instanceGeometry?.FirstOrDefault(x => x is Curve) as Curve;
var instanceReference = instanceCurve?.Reference;
var instanceRepresentation = instanceReference?.ConvertToStableRepresentation(_doc);
var symbolGeometry = geometryInstance?.GetSymbolGeometry();
var symbolCurve = symbolGeometry?.FirstOrDefault(x => x is Curve) as Curve;
var symbolReference = symbolCurve?.Reference;
var symbolRepresentation = symbolReference?.ConvertToStableRepresentation(_doc);
```
Executing this provides the following values:
- `instanceRepresentation` = e558d96b-a4b0-449d-a84e-00d8c2768a5c-00000a38:2:1:LINEAR
- `symbolRepresentation` = e558d96b-a4b0-449d-a84e-00d8c2768a5c-00000c0f:0:INSTANCE:e558d96b-a4b0-449d-a84e-00d8c2768a5c-00000a38:2:1:LINEAR
These include two `UniqueId` values:
- e558d96b-a4b0-449d-a84e-00d8c2768a5c-00000a38 – `CADLinkType`
- e558d96b-a4b0-449d-a84e-00d8c2768a5c-00000c0f – `ImportInstance`
For me, it seems like these two results are mixed up.
Looks like `instanceRepresentation` refers to symbol geometry, while `symbolRepresentation` refers to instance geometry.
\*\*Answer:\*\* Seems logical in a way, no?
When you use symbol, it gives the full lineage of symbol and instance of that symbol.
When you use the copy (with method noted below), it just gives the symbol it was copied from.
There is no actual instance for it, because that function just creates a copy at the time for you (is a helper method for specific purposes).
Beyond CADLinks, you'll find that there are multiple versions of symbol geometries for a type, i.e., there is often a symbol to represent each structural framing length (with such lengths being driven by instance variations, not type variations).
So, equating symbol geometry to family symbols probably is confusing to start with.
That is to say they are all different ids and at time of extraction the form of symbol geometry you get is going to be partly decided by the instance variations not just the type variations.
Extract from [RevitAPI.chm on GeometryInstance.GetInstanceGeometry](https://www.revitapidocs.com/2022/22d4a5d4-dfc2-7227-2cae-b989729696ec.htm):
> ...This method returns a copy of the Revit geometry. It is suitable for use in a tool which extracts geometry to another format or carries out a geometric analysis; however, because it returns a copy the references found in the geometry objects contained in this element are not suitable for creating new Revit elements referencing the original element (for example, dimensioning). Only the geometry returned by GetSymbolGeometry() with no transform can be used for that purpose."
Here is a simple example demonstrating these relationships, snooping the elements with RevitLookup:
![Two beams of same family type](img/rpt_symb_vs_inst_geom_1.png "Two beams of same family type")

Two beams of same family type

The short beam element id is 427840:
![The short beam as id 427840](img/rpt_symb_vs_inst_geom_2.png "The short beam as id 427840")

The short beam as id 427840

The long beam element id is 427855:
![The long beam as id 427855](img/rpt_symb_vs_inst_geom_3.png "The long beam as id 427855")

The long beam as id 427855

The `FamilySymbol` element id is 95037:
![The FamilySymbol id 95037](img/rpt_symb_vs_inst_geom_4.png "The FamilySymbol id 95037")

The FamilySymbol id 95037

If you check the bounding box extents for the two geometry symbols, you'll see they match the beam lengths.
This gets further complicated with cuts, but it demonstrates that Revit is storing geometrical symbol variations differently to how we think of the type-to-instance relationships based on type and instance parameters.
Many thanks to Richard for yet another insightful and illuminating explanation!
#### Create New Material with Texture
Harm van den Brand shares a new implementation of a suggestion by Rudi \*Revitalizer\* Honke to create a new material and set its texture in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [importing and displaying satellite images](https://forums.autodesk.com/t5/revit-api-forum/importing-and-displaying-satellite-images/m-p/10815534):
I'm building an add-in for Revit, and I would like to be able to import and display third-party satellite imagery in order to place buildings in their 'real' position.
I would like to be able to do this in a 3D view, but I don't know how.
The user workflow for my add-in is this:
- A user opens the add-in and is prompted to input a location through a WPF window.
- Once a location is confirmed, a number of things are created/imported into the active project to make it look as close as possible to its actual real-life location. One of these things is the satellite image I'm seeking to import here.
Essentially, my question is exactly [this one](), but instead doing that programmatically/automatically through an add-in. In that thread, a suggestion is made to create a decal with the desired image, but this does not seem to be supported through the API.
Another approach I found is to use `PostCommand` to create and place decals, but these commands are apparently only executed after exiting the API context and only one at a time.
As my add-in aims to perform a whole bunch of functionalities in one go, this seems ill-suited for my use case.
It seems to be possible to chain a bunch of `PostCommand` calls, but this is a little 'hacky' and not recommended, especially for commercial use.
Am I overlooking some existing functionality?
Is my use case just not supported in current Revit?
I'm new to programming for Revit, so it's very possible I've missed something.
I'm running / programming for Revit 2019 on Windows 10.
\*\*Answer:\*\* What about creating a new material and setting its texture path as described
in [modifying material visual appearance](https://thebuildingcoder.typepad.com/blog/2017/11/modifying-material-visual-appearance.html),
then making a `TopoSurface` and assigning the material to it?
I don't know how to adjust the UV mapping for the TopoSurface, but if it worked, you would see your satellite image in 3D.
\*\*Response:\*\* Thanks to all for the replies!
It took some time to try out the proposed solution (accessing AppearanceElements is convoluted!), so that's why it took me this long to reply.
In the end, though I had to work around some weird quirks with the API.
Adding the image as a texture to a topography through a material works great.
I ended up taking Revitalizer's suggested approach of creating a new material and setting its texture.
```csharp
Material underlayMaterial = Material.Create(
revitDocument, materialName);
```
To this material, I link a so-called `AppearanceAsset`:
```csharp
underlayMaterial.AppearanceAssetId = assetElement.Id;
```
(more on how I get this assetElement in a moment)
Then, I assign the path of a jpeg image to the texture asset:
```csharp
using (AppearanceAssetEditScope editScope
= new AppearanceAssetEditScope(revitDocument))
{
Asset editableAsset = editScope.Start(
assetElement.Id);
AssetProperty assetProperty = editableAsset
.FindByName("generic_diffuse");
Asset connectedAsset = assetProperty
.GetConnectedProperty(0) as Asset;
// Edit bitmap
if (connectedAsset.Name == "UnifiedBitmapSchema")
{
AssetPropertyString path = connectedAsset
.FindByName(UnifiedBitmap.UnifiedbitmapBitmap)
as AssetPropertyString;
if (path.IsValidValue(imagePath))
path.Value = imagePath;
// You might have to fiddle a bit with the scale properties,
// for example when your source uses centimeters:
AssetPropertyDistance scaleX = connectedAsset
.FindByName(UnifiedBitmap.TextureRealWorldScaleX)
as AssetPropertyDistance;
AssetPropertyDistance scaleY = connectedAsset
.FindByName(UnifiedBitmap.TextureRealWorldScaleY)
as AssetPropertyDistance;
// Because newly added bitmaps are displayed in inches.
if (scaleX.DisplayUnitType
== DisplayUnitType.DUT_DECIMAL_INCHES)
scaleX.Value /= 2.54;
if (scaleY.DisplayUnitType
== DisplayUnitType.DUT_DECIMAL_INCHES)
scaleY.Value /= 2.54;
}
editScope.Commit(true);
}
```
I find this a bit 'hacky' because of how I retrieve the assetElement.
Instinctively, I would want to create a new, empty instance of Asset.
Something like:
```csharp
AppearanceAssetElement assetElement
= AppearanceAssetElement.Create();
```
However, this is not how Revit's material/texture API works.
We can only use those materials/textures/etc. that are present in Revit's libraries.
Therefore, we can only make copies of existing ones:
```csharp
// Retrieve asset library from the application
// (this is the only source available;
// instantiating from zero is impossible)
IList assetList = commandData.Application
.Application.GetAssets(AssetType.Appearance);
// Select arbitrary asset from library
// (200 works, not all do)
Asset asset = assetList[200];
AppearanceAssetElement assetElement;
try
{
assetElement = AppearanceAssetElement
.Create(revitDocument, someNewName, asset);
}
```
Yes, I really just randomly tried indices in that assetList until I found one that worked, and hardcoded that one in. Not all AppearanceAssets in the list have the necessary "generic_diffuse" assetProperty to which we can bind a texture, so we have to select one that does.
If you are developing your addin for external parties, this is risky, because we can't ensure that the same libraries are available for any particular user.
It's probably best to somehow filter for valid AppearanceAssets.
Also, you can see that retrieving this appearanceAsset requires `ExternalCommandData` (which I named CommandData in the code given), which an addin retrieves via the 'Execute' method of an `IExternalCommand` implementing class.
Also, remember to wrap most of these snippets in transactions.
I hope this helps!
Many thanks to Revitalizer for the good suggestion and to Harm for the implementation and notes.
#### Updates from Devteam, Richard, and Harm
Richard adds: You can find an asset that contains a certain named property via Linq e.g.:
```vbnet
Dim Assets As List(Of Visual.Asset) = app.Application.GetAssets(Visual.AssetType.Appearance)
' Find an asset with a property matching Visual.Generic.GenericDiffuse
Dim J0 = Assets.FirstOrDefault(Function(x) x.FindByName(Visual.Generic.GenericDiffuse) IsNot Nothing)
```
This seems better than a fixed index since you don't know for sure how that number may change, e.g., if user manually adds an asset etc.
Obviously, the Assets library is quite large to search but there are existing Materials and Assets in the document which can be duplicated instead.
So, it is often better to know what you are looking for, e.g., load a Family with your known material and the Asset you want to manipulate (if you can't find it).
Most materials will have the appearance asset with the bitmap property though.
There is an example of this within `RevitAPI.chm` under `AppearanceAssetElement.Duplicate`.
As noted, if it isn't there in the document then it is straightforward to load something with it to add it.
The development team also responded, saying:
An asset is a resource with economic value that an individual, corporation or country owns or controls with the expectation that it will provide a future benefit.
In Revit, an (material) asset is owned by the libraries too, but currently the libraries are only data but not classes in code, so there's no way to create an asset from the libraries, but can only duplicate one from existing.
The user's problems is trying to find an asset that can assign an image; I suggest below steps:
- Duplicate a new asset (or material) from any existing ones that serves only for this purpose.
- If the new asset contains "unified_bitmap", then change it as desired.
- If not, then create a connected asset with AssetProperty.AddConnectedAsset() that should contains "unified_bitmap" in the schema.
At last, exposing decal element to API should be the best way. (There may also contains some workaround or hacky way through external file mechanism, but I cannot confirm that.)
Harm responds:
Thanks for the update.
If I understand correctly, my solution follows steps 1 and 2 of the dev-team's suggestion.
So it's good to know I'm not doing anything too weird.
Except for selecting a hardcoded element from the library by hand, but I'll happily use @RPTHOMAS108 's suggestion of selecting one through list filtering.
It'll prevent any errors that would occur if the asset libraries ever were to change.
Always good to go through old code again and find yourself thinking "this could be improved".
I'm satisfied with how I currently create/copy an asset (again, step 1 and 2).
It's just that when I first started digging through the api to get to the functionality I needed, it was a bit like groping around in the dark.
My first instinct when it came to the `AppearanceAsset` part of the problem, at the time, was to instantiate a new one.
It wasn't immediately clear to me that I couldn't do that, but that I could duplicate one.
I learn more with every feature I implement :-)
That having been said, things like your blog and the documentation at revitapidocs have helped out immensely, with this problem and with others, so thank you very much!
#### RVT Dashboard Data Access
Some notes from an internal discussion on how to access data in RVT for a dashboard:
\*\*Question:\*\* I would like to collect data from Revit models for display in a dashboard.
I thought of using the model derivative APIs in Forge to retrieve the data, or use DA4R since the Revit model must be opened to access the database.
However, I do not have access to Forge or DA4R and would prefer a way to read the data without Revit.
I assume that Revit must be used to unpack the database to make it useable to collect data from.
Much of the content in Revit is created on demand dynamically and not necessarily stored in the file database.
Am I on the right track here?
I heard about some other app that can collect the Revit model data needed.
How can it read the contents without Revit?
Is there some cloud service utilizing DA4R or similar to process the model?
\*\*Answer:\*\* The RVT file format is a structured storage file, so some content can be pulled without opening the file directly.
Because of that format, some data is quite easy to access, e.g., using transmission data and basic file info, both of which can be read with tools like Structured Storage Viewer:
- [BasicFileInfo](https://www.revitapidocs.com/2022/475edc09-cee7-6ff1-a0fa-4e427a56262a.htm)
- [TransmissionData](https://www.revitapidocs.com/2022/d78d1e9c-1cee-1336-88d5-b605dacd077d.htm)
Other aspects are more difficult, e.g., how many light fixtures are in room number 2143, how many warnings are in the model, how many doors don't have a valid mark, etc.
Forge isn't necessarily a 'must use' as the data might be accessible via other means, e.g., upload to BIM360 or use the online viewer; use the model checker, or move to another file format for the final deliverable.
Also, might be able to mine data from the digital twin or an IFC instead of an RVT.
Short of those two, it's likely best to stick with forge.
#### Marking and Retrieving a Custom Element
\*\*Question:\*\* My add-in creates its own view to show additional data within Revit.
I hit a bump around the view element id and wonder whether anything in the API might be able to help.
I essentially want to create my own temporary view within Revit to show data that is not from the currently loaded model, i.e., from linked files or elsewhere.
Right now, I create a view, populate it and delete it on shutdown.
This is all fine.
However, if the view is the only one open in Revit on shutdown, it gets saved into the file.
This got me thinking that I could just save a default add-in view and look for it on next load.
However, I can't find a way to determine the view element id, so I don't know what to look for.
- Can I create a view with a read-only name, so the user can't edit it and I can search for that?
- Can I define a Revit view ID using a GUID somehow?
Is there anywhere I could store the view ID, so that I can retrieve it on load?
I considered storing it in my own settings file, but that doesn't work if the file gets sent to another user.
\*\*Answer:\*\* This is a prime case for [extensible storage](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.23) in my opinion.
Make a new schema and save the `UniqueId` of the view into it.
Users will delete that view, though, and if it doesn’t file into the project browser correctly, there will be push back.
Expect to delete and recreate the view often (even mid-session).
Also, ensure that you have good product documentation on why this is in the file, and how it can be worked with, and the like.
Otherwise you will have a LOT of support cases around the feature.
\*\*Response:\*\* Fantastic.
Yes, documentation and the options we present to the user around how they use this feature are important.
We also need to be careful around the default name of the view, so it's purpose is obvious enough.
Thanks for the help and advice.
#### Advanced Remote Batch Command Processing
Several people recently raised questions on automating Revit workflows, and one possibility to consider is the use of Revit journal files.
David Echols, Senior Programmer at Hankins & Anderson, Inc. shared some important insights and experience in this area in his Autodesk University 2014 class SD5980 on \*Advanced Revit Remote Batch Command Processing\*:
> This class explains a process to run external commands in batch mode from a central server to remote Revit application workstations.
It covers how to use client and server applications that communicate with each other to manage Revit software on remote workstations with WCF (Windows Communication Foundation) services, examines how to pass XML command data to the Revit application to open a Revit model and initiate batch commands, shows a specific use case for batch export of DWG files for sheets, examines a flexible system for handling Revit dialog boxes on the fly with usage examples and code snippets, and discusses the failure processing API in the context of bypassing warning and error messages while custom commands are running. Finally, it shows you how to gracefully close both the open Revit model and the Revit application.
> - [Handout: \*RevitJournals.pdf\*](zip/RevitJournals.pdf)
#### Midwinter Break
We are nearing the middle of winter here on the northern hemisphere, and Autodesk is celebrating company holidays in the last days of the calendar year.
I am looking forward to some peaceful time to recuperate during the end
of [advent](https://en.wikipedia.org/wiki/Advent),
followed by the [twelve-night](https://en.wikipedia.org/wiki/Twelfth_Night_(holiday)) turning point of the year,
also known as [Rauhnächte](https://de.wikipedia.org/wiki/Rauhnacht) or \*raw nights\* in German,
full of special depth and significance, related to the differences between
the [lunar](https://en.wikipedia.org/wiki/Lunar_calendar) and solar cycles,
beginning with [Christmas](https://en.wikipedia.org/wiki/Christmas),
[Hanukkah](https://en.wikipedia.org/wiki/Hanukkah),
Celtic [Samhain](https://en.wikipedia.org/wiki/Samhain),
Druid [Alban Arthan](https://en.wikipedia.org/wiki/Alban_Arthan),
and many other sacred traditions.
A time of confusion, breaking things, going wrong, calming down, going slowly, contemplation, relaxing into peace and quiet and new beginnings.
I wish you a wonderful midwinter break full of light and warmth!
![Candlelight in snow](img/candlelight_snow.jpg "Candlelight in snow")