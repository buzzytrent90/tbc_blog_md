---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.3
content_type: code_example
optimization_date: '2025-12-11T11:44:13.969737'
original_url: https://thebuildingcoder.typepad.com/blog/0452_2011_subscription.html
post_number: '0452'
reading_time_minutes: 5
series: general
slug: 2011_subscription
source_file: 0452_2011_subscription.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- levels
- parameters
- references
- revit-api
- selection
- sheets
- vbnet
- views
title: Subscription Release and Updated SDK
word_count: 993
---

### Subscription Release and Updated SDK

Lots of really important news coming in right now, both because the mid-year updates of the Revit family products are now available and because we just released the
[Revit 2011 DevTV](http://thebuildingcoder.typepad.com/blog/2010/09/revit-2011-web-update-2.html) recordings that we are so excited about.

Similar to last year, Autodesk provides two types of mid-year updates to Revit:

- The
  [Web Update 2 (WU2)](http://thebuildingcoder.typepad.com/blog/2010/09/revit-2011-web-update-2.html) that I recently discussed.- The Subscription Release.

Unlike the public web updates, the Revit 2011 subscription releases are available only for
[subscription customers](http://www.autodesk.com/subscriptionlogin).

#### Updated SDK

Both the web update and the subscription releases include a number of API enhancements.
Last year, there were separate sets of updated SDKs for the WU2 and subscription release.
This year, Autodesk provides one single updated SDK.
Certain features are available only in the subscription release.
They are clearly marked in the API reference documentation.

The updated SDK has been posted to the
[Revit Developer Center](http://www.autodesk.com/developrevit).
Here is a direct link as well, for your convenience:

- [Revit 2011 SDK (Update Sept. 2010)](http://images.autodesk.com/adsk/files/revit2011sdk0.exe) (exe - 108143Kb)- [Readme - Revit SDK 2011 SDK (Update Sept. 2010)](http://images.autodesk.com/adsk/files/readme_-_revit_2011_sdk_(update__sept_22__2010)_devcenter.docx) (docx - 19Kb)

The product updates themselves do not include the updated SDK, so you have to download it separately.

In addition to the API enhancements, the SDK samples and the Revit Lookup tool have been updated.

Here is a summary of API changes and enhancements made in these releases:

- [Issues addressed](#2)- [New Features](#3)
    - [Conceptual Energy Analysis API](#4)- [City Time Zone](#5)- [Document Preview](#6)- [Changes to the 2011 SDK](#7)

#### Issues addressed

- Attributes of Revit API DLLs need to be updated (190870)- Autodesk.Revit.DB.Domain is missing enum items for Conduit, CableTray (190981)- Accessing grids using selection through the API causes stability problem after closing Revit (190867)- ElectricalSystem.AddToCircuit returns true for a Spare circuit, but doesn't actually connect (191518)- RaytraceBounce sample: model curves along rays not created because curve is not in sketchplane (192058)- Stability problem when select to save a family (191578)- SDK sample MaterialQuantities does not successfully calculate Gross material quantities (192331)- DetailLine/AnnotationSymbol cannot be created in ViewSheet programmatically (No sketch plane) (192072)- API issue: Save /SaveAs lose the preview image of family file (192545)- Detail curve cannot be create in a plan view via NewDetailCurve in a specific plan view (192115)- Setting formula for Manufacturer/Type Comments as string using the API throws exception (192866)- New electrical shared parameter types are missing (192832)- CableTray Connector Angle property throws exception (190873)- Family Type count using the API is inconsistent with UI (192889)

#### New Features

In addition to the enhancements listed above, it includes the following new API features:

- Conceptual Energy Analysis API (Subscription only)- City Time Zone- Document Preview

#### Conceptual Energy Analysis API

Conceptual Energy Analysis API is a major enhancement to the Revit API. This feature will be functional only in sessions with licensed access to the Revit 2011 Subscription Advantage Pack. The following classes are added in the Autodesk.Revit.DB.Analysis namespace and provide access to the elements and objects created by Revit to perform energy analyses on conceptual design models:

- ConceptualConstructionType- ConceptualSurfaceType- MassEnergyAnalyticalModel- MassLevelData- MassSurfaceData- MassZone

The following method supports the export of a gbXML file containing conceptual energy analysis elements (mass elements) only.

- Document.Export(string,string,MassGBXMLExportOptions)

#### City Time Zone

The following new static method provides direct access to the results of a time zone calculation for a given longitude and latitude:

- SunAndShadowSettings.CalculateTimeZone()

The City class has been adjusted to return more accurate TimeZone values than in the previous implementation.
Note that the properties, SiteLocation.Latitude and SiteLocation.Longitude, do not adjust the time zone of the site location according to these calculations. But you can make this change directly by setting the TimeZone property.

#### Document Preview

The following new methods are added to allow you to access the settings related to the stored document preview image for a given document:

- Document.GetDocumentPreviewSettings this returns a DocumentPreviewSettings object- DocumentPreviewSettings.PreviewViewId - the id of the view to use for the preview- DocumentPreviewSettings.ForceViewUpdate() - sets the document to update the view before saving (useful when the document is never displayed)

The following code snippet shows a sample use of the enhancement described above:
```csharp
  // ...
  View3D view3d
    = d.FamilyCreate.NewView3D(
      new XYZ(1, 2, 0).Normalize() );

  // Assign permanent preview id

  DocumentPreviewSettings dps
    = d.GetDocumentPreviewSettings();

  dps.PreviewViewId = view3d.Id;
  dps.ForceViewUpdate( true );

  t.Commit();
  d.SaveAs( @"C:\Door-Automated.rfa" );
  // ...
```

#### Changes to the 2011 SDK

- RevitAPI.chm- SDK sample ArchSample is updated: due to category length beyond the limitation of excel sheet name length. - Samples/ArchSample/VB.NET/Command.vb- SDK sample MaterialQuantities did not calculate Gross material quantities correctly - Samples/MaterialQuantities/CS/MaterialQuantities.cs- SDK sample RaytraceBounce updated to generate curves properly. - Samples/FindReferencesByDirection/RaytraceBounce/CS/RayTraceBounceForm.cs- Fixed sample bugs and correct typos, formats
          - Samples/DirectionCalculation/CS/DirectionCalculation.addin#2 integrate- Samples/GenerateFloor/CS/GenerateFloor.addin#2 integrate- Samples/ImportExport/CS/Import/ImportDWGData.cs#4 integrate- Samples/ImportExport/CS/Import/ImportGBXMLData.cs#3 integrate- Samples/Massing/ParameterValuesFromImage/CS/ParameterValuesFromImage.csproj#2 edit- Samples/Massing/ParameterValuesFromImage/ParameterValuesFromImage.suo#2 delete- Samples/Massing/ParameterValuesFromImage/ParamValueFromImage.suo#2 delete- Samples/Revit 2011 New Samples.doc#5 integrate- Samples/RvtSamples/CS/RvtSamples.txt#4 edit- Samples/SamplesContent.htm#4 edit- Samples/SDKSamples2011.sln#4 integrate- Samples/Selections/CS/Command.cs#4 integrate- Samples/Selections/CS/SelectionFilters.cs#3 integrate- Samples/Selections/CS/SelectionForm.cs#3 integrate- Samples/Selections/CS/SelectionForm.Designer.cs#2 integrate- Samples/Selections/CS/SelectionManager.cs#3 integrate- Samples/Selections/CS/Selections.addin#2 integrate- 190662: RevitLookup snoop DB lacks the ElementType node:- Add sfm.Clear() and GetChangeTypeElementDeletion trigger
              Samples/AnalysisVisualizationFramework/DistanceToSurfaces/CS/Command.cs