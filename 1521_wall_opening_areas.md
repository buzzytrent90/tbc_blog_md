---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.4
content_type: qa
optimization_date: '2025-12-11T11:44:16.209341'
original_url: https://thebuildingcoder.typepad.com/blog/1521_wall_opening_areas.html
post_number: '1521'
reading_time_minutes: 9
series: general
slug: wall_opening_areas
source_file: 1521_wall_opening_areas.md
tags:
- csharp
- elements
- family
- filtering
- levels
- references
- revit-api
- rooms
- sheets
- transactions
- walls
- windows
title: Wall Opening Areas
word_count: 1846
---

### Family Category and Two Energy Model Types
Let's pick up two more topics from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) today:
- [Family `Category` property is not always set](#2)
- [Two different energy analysis model types](#3)
- [gbXML export options](#4)
- [`BuildingEnvelopeAnalyzer` class](#5)
- [`EnergyAnalysisDetailModel` creation from building elements and volumes](#6)
- [`EnergyAnalysisDetailModelOptions`](#7)
- [Quo Vadis Revit API QAS?](#8)
#### Family Category Property is not Always Set
A well-known issue that has been around forever cropped up again yesterday in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
on [having trouble filtering to `OST_TitleBlocks`](http://forums.autodesk.com/t5/revit-api-forum/having-trouble-filtering-to-ost-titleblocks/m-p/6827759):
The filter was set up like this:
```csharp
FilteredElementCollector collector
= new FilteredElementCollector( doc );
ICollection titleFrames = collector
.OfCategory( BuiltInCategory.OST_TitleBlocks )
.OfClass( typeof( Family ) )
.ToElements();
```
Internally, this filters for all `Family` elements and queries their `Category` property.
Unfortunately, the `Category` property is not always implemented on families, so you may have to use `FamilyCategory` instead.
Sometimes, that is not set either.
In that case, you can use the category of the family's first symbol.
That is mentioned in the discussion
on [Changing Element Type](http://thebuildingcoder.typepad.com/blog/2015/09/change-type-iterate-elements-create-family.html#2),
and the workaround querying the family's first symbol for its category is demonstrated in
the [ADN Xtra labs](https://github.com/jeremytammik/AdnRevitApiLabsXtra)
module [Labs3.cs](https://github.com/jeremytammik/AdnRevitApiLabsXtra/blob/master/XtraCs/Labs3.cs)
in [lines 86-118](https://github.com/jeremytammik/AdnRevitApiLabsXtra/blob/master/XtraCs/Labs3.cs#L86-L118).
For future reference, I implemented two new helper methods `FamilyFirstSymbolCategoryEquals` and `GetFamiliesOfCategory`
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) to
demonstrate this, in the
module [CmdCollectorPerformance.cs, lines 292-L330](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs#L292-L330):
```csharp
static bool FamilyFirstSymbolCategoryEquals(
Family f,
BuiltInCategory bic )
{
Document doc = f.Document;
ISet ids = f.GetFamilySymbolIds();
Category cat = (0 == ids.Count)
? null
: doc.GetElement( ids.First() ).Category;
return null != cat
&& cat.Id.IntegerValue.Equals( (int) bic );
}
static void GetFamiliesOfCategory(
Document doc,
BuiltInCategory bic )
{
IEnumerable families
= new FilteredElementCollector( doc )
.OfClass( typeof( Family ) )
.Cast()
.Where( f =>
FamilyFirstSymbolCategoryEquals( f, bic ) );
}
```
In some cases, due to this limitation and depending on what you wish to achieve, it might be handier to filter for family symbols or family instances rather than the top-level families themselves.
#### Two Different Energy Analysis Model Types
Another issue deals with invalid window dimensions caused
by [gbXml export using energy settings](http://forums.autodesk.com/t5/revit-api-forum/gbxml-export-using-energy-settings/m-p/6784904):
\*\*Question:\*\* I am exporting gbXML files using the Revit API. This seems to be using the Revit energy settings. Revit breaks up the wall the window is located within by creating a wall the same size as the window and then creates a mesh around it (see picture). When I translate the gbXML file to an EnergyPlus Idf file and run the analysis, EnergyPlus throws an error because the window is the same size as the wall it is located within.
When I use the Revit gbXML export dialog and export Room/Space volumes this is not the case.
It exports the window located within the wall as drawn in Revit.
I do however want to use the API to export. Am I getting it wrong somewhere and is there a work around or is this a bug?
![One window exported to gbXML](img/gbxml_onewindow.png)
Devon Powell suggested working around this by changing the `ExportEnergyModelType` to `SpatialElement`:
> You can use the API to use the spaces export method to export the gbXML file (not sure if you want to use this, or you are specifically trying to use the energy settings method.) You first need to setup your energy model settings then create the energy model. Then you can export the gbXML file. See example code below.
```vbnet
'define the energy model options
Dim EmOpt As New EnergyAnalysisDetailModelOptions
EmOpt.EnergyModelType = EnergyModelType.SpatialElement
EmOpt.ExportMullions = False
EmOpt.IncludeShadingSurfaces = True
EmOpt.SimplifyCurtainSystems = True
'create the energy analysis (modifies model, wrapped in tx)
Using tx As New Transaction(doc)
tx.Start("Create Internal E-Model")
Dim Em As EnergyAnalysisDetailModel = EnergyAnalysisDetailModel.Create(doc, EmOpt)
tx.Commit()
End Using
'set the gbXML options
Dim gbxmlOpt As New GBXMLExportOptions
gbxmlOpt.ExportEnergyModelType = ExportEnergyModelType.SpatialElement
'export the gbXML file
doc.Export(folder, doc.Title, gbxmlOpt)
```
Jaco Kemp confirms that this helps and shares the corresponding C# code:
> Thank you for the example. It works for exporting spaces and don't have the same problem as exporting surfaces. This will do for now. For completeness, I have included the C# code of your solution.
```csharp
public static string RevitToGbXMLSpaces( string projectLocation )
{
//define the energy model options
var EmOpt = new EnergyAnalysisDetailModelOptions();
EmOpt.EnergyModelType = EnergyModelType.SpatialElement;
EmOpt.ExportMullions = true;
EmOpt.IncludeShadingSurfaces = true;
EmOpt.SimplifyCurtainSystems = true;
//create the energy analysis (modifies model, wrapped in tx)
Document doc = DocumentManager.Instance.CurrentDBDocument;
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Create Internal E-Model" );
EnergyAnalysisDetailModel.Create( doc, EmOpt );
tx.Commit();
}
//set the gbXML options
var gbxmlOpt = new GBXMLExportOptions()
{
ExportEnergyModelType = ExportEnergyModelType.SpatialElement
};
//export the gbXML file
string fileName = "temp";
doc.Export( projectLocation, fileName, gbxmlOpt );
return projectLocation + "\" + fileName + ".xml";
}
```
This topic also arose in a follow-up conversation after the extensive discussion on determining wall opening areas per room, when some of the walls may not be vertical:
- [Determining wall cut area for a specific room](http://thebuildingcoder.typepad.com/blog/2016/04/determining-wall-cut-area-for-a-specific-room.html)
- [More on wall opening areas per room](http://thebuildingcoder.typepad.com/blog/2016/04/more-on-wall-opening-areas-per-room.html)
Sample code handling room volumes extending sideways at different elevations was presented at AU 2014.
There are two different ways to determine the energy calculation volumes:
One is based on either the floor plan (for vertical faces), the other on volumetric filling up of the spaces with voxels (for slanted faces).
The choice between the two methods is made by setting the `EnergyAnalysisDetailModelOptions.EnergyModelType` property, which can take one of the following two enumeration values:
- `SpatialElement` – Energy model based on rooms or spaces.
- `BuildingElement` – The building element based energy analytical model.
You can see this switch in the Export gbXML dialogue, and the enhancement is mentioned in
the [What's New in the Revit 2015 API](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html) documentation on
[Energy analysis API additions](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#3.12):
#### gbXML Export Options
The new property `GBXMLExportOptions.ExportEnergyModelType` determines the type of analysis used when producing the export gbXML file for the document. Options are:
- SpatialElement – Energy model based on rooms or spaces. This is the default for calls when this option is not set, and matches behaviour in Revit 2014.
- BuildingElement – Energy model based on analysis of building element volumes.
#### BuildingEnvelopeAnalyzer Class
The new class `BuildingEnvelopeAnalyzer` analyses which elements are part of the building envelope (the building elements exposed to the outside). This class uses a combination of ray-casting and flood-fill algorithms in order to find the building elements that are exposed to the outside of the building. This method can also look for the bounding building elements for enclosed space volumes inside the building. Options for the analysis include:
- AnalyzeEnclosedSpaceVolumes – Whether or not to analyse interior connected regions inside the building forming enclosed space volumes.
- GridCellSize – The cell size for the uniform cubical grid used when analysing the building envelope.
- OptimizeGridCellSize – Whether or not to use the exact value for the cell size or let the analyser optimize the cell size based on the specified grid size
#### EnergyAnalysisDetailModel Creation from Building Elements and Volumes
[What's New in the Revit 2016 API](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html) mentions
further [energy analysis and gbXML API changes](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html#4.06):
The function `EnergyAnalysisDetailModel.Create` now offers the ability to create energy model based on analysis of building element boundaries and volumes (set `EnergyAnalysisDetailModelOptions.EnergyModelType` to `BuildingElement`). This option matches the default energy model creation used by Revit's user interface.
The generated energy model is affected by settings in `EnergyDataSettings`, including the option to use the new enumerated value `AnalysisMode.ConceptualMassesAndBuildingElements`.
This option sets the generation of the `EnergyAnalysisDetailModel` to use the combination of conceptual masses and building elements.
#### EnergyAnalysisDetailModelOptions
The new property `EnergyAnalysisDetailModelOptions.EnergyModelType` indicates whether the energy model is based on rooms/spaces or building elements. Options are:
- SpatialElement – Energy model based on rooms or spaces. This is the default for calls when this option is not set, and matches behaviour in Revit 2015.
- BuildingElement – Energy model based on analysis of building element volumes.
#### Quo Vadis Revit API QAS?
This is a note to myself and my colleagues on my on-going research to implement a question answering system to handle the closed-domain topic of \*Getting started with the Revit API\*.
I would really appreciate some guidance on which system to start working seriously with and where to go next.
Here are some systems I took a look at so far:
- [Jill Watson](http://thebuildingcoder.typepad.com/blog/2017/01/au-in-london-and-deep-learning.html#7)
- [IBM Watson and Bluemix](http://thebuildingcoder.typepad.com/blog/2017/01/virtues-of-reproduction-research-mep-settings-ontology.html#6)
- [Microsoft QnA Maker](http://thebuildingcoder.typepad.com/blog/2017/01/vertical-dimensioning-and-revit-api-qas-research.html#4)
- [Open Source QAS Options](http://thebuildingcoder.typepad.com/blog/2017/01/virtues-of-reproduction-research-mep-settings-ontology.html#7)
- [Stanford DARPA DeepDive](http://thebuildingcoder.typepad.com/blog/2017/01/vertical-dimensioning-and-revit-api-qas-research.html#5)
- [YodaQA and DL-Learner](http://thebuildingcoder.typepad.com/blog/2017/01/vertical-dimensioning-and-revit-api-qas-research.html#6)
- [TensorFlow and Keras](http://thebuildingcoder.typepad.com/blog/2017/01/textnote-rotation-forge-devcon-tensorflow-and-keras.html#4)
Watson and QnA Maker looks simplest to get started with.
I would much prefer to use an open source system, though, and avoid everything proprietary.
I summarised my thoughts on what I would like to achieve and what material I can use to build a knowledge base and teach the system:
- [More Research on a Revit QAS](http://thebuildingcoder.typepad.com/blog/2017/01/vertical-dimensioning-and-revit-api-qas-research.html#3)
- [Building a Revit API Ontology](http://thebuildingcoder.typepad.com/blog/2017/01/virtues-of-reproduction-research-mep-settings-ontology.html#8)
Here are my main open questions ten days ago:
- [My Current Open Questions on Question Answering Systems](http://thebuildingcoder.typepad.com/blog/2017/01/vertical-dimensioning-and-revit-api-qas-research.html#7)
Nothing much has changed since then.
I still wonder:
- Which system should I choose?
- Where should I go next?