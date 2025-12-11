---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: news
optimization_date: '2025-12-11T11:44:16.152265'
original_url: https://thebuildingcoder.typepad.com/blog/1493_au_2017_1_sdk_freeze.html
post_number: '1493'
reading_time_minutes: 6
series: general
slug: au_2017_1_sdk_freeze
source_file: 1493_au_2017_1_sdk_freeze.md
tags:
- elements
- family
- geometry
- levels
- parameters
- references
- revit-api
- selection
- sheets
- views
title: The Building Coder
word_count: 1240
---

### AU, Revit 2017.1 SDK and REX FreezeDrawing
I returned from
the [Munich Forge accelerator](http://autodeskcloudaccelerator.com),
travelling back to Switzerland by train.
For ecological reasons, I prefer to avoid flying whenever I possibly can.
Last Friday was the deadline for submitting my Autodesk University handout, so that kept me busy.
I still need to improve it a bit, and I hope a post-deadline update is feasible.
I am presenting two classes, and I have a special discount to offer you.
Lots of other things are happening as well.
Here is the list of topics for today:
- [My classes at Autodesk University](#2)
- [$400 AU registration discount](#3)
- [Revit 2017.1 SDK](#4)
- [REX SDK FreezeDrawing sample](#5)
- [Determining the height of a column](#6)
#### My Classes at Autodesk University
This year, I am presenting two classes at [Autodesk University](http://au.autodesk.com/las-vegas/overview):
- SD20891 – Revit API Expert Roundtable – Open House on the Factory Floor
- SD20908 – Connect Desktop and Cloud – Free Your BIM Data!
One way to access them and check out their details is
to [search the AU catalogue for 'tammik'](https://events.au.autodesk.com/connect/dashboard.ww#loadSearch-searchPhrase=tammik&searchType=session).
I really hope you can make it to my live sessions, of course.
In case not, I'll share my handouts for both as soon as I have cleaned them up a little bit more.
Meanwhile, I have another AU goodie to share, in case you have not already discovered it elsewhere:
#### $400 Autodesk University Registration Discount
I have a special offer to share with you:
You can get $400 off the regular AU conference price of $2,175 on any new registration.
Simply use the discount code `400AU16REFSP`.
Here is [the AU registration link](https://events.au.autodesk.com/portal/login.ww).
#### Revit 2017.1 SDK
As you probably know by now, Revit 2017.1 has been released, with an [impressive list of enhancements](https://knowledge.autodesk.com/support/revit-products/learn-explore/caas/CloudHelp/cloudhelp/2017/ENU/Revit-WhatsNew/files/GUID-E287E537-7122-4ED8-8845-B8E24F287B62-htm.html).
An updated Revit SDK for Revit 2017.1 has been posted to
the [Revit Developer Centre](http://www.autodesk.com/developrevit):
- [Revit 2017.1 SDK Update Oct 19, 2016.msi](http://images.autodesk.com/adsk/files/Revit_2017.1_SDK_(Update_Oct_19__2016).msi) (msi – 301220Kb)
The new version sports quite a large list of exciting API improvements, as you can see by looking at the extensive documentation of them included with the SDK, both in the stand-alone document \*Revit Platform API Changes and Additions.docx\* and the section on \*What's New\* in the Revit API help file `RevitAPI.chm`.
It also includes a new REX SDK sample:
#### REX SDK FreezeDrawing Sample
One important structural part of the Revit SDK is the [REX SDK](http://thebuildingcoder.typepad.com/blog/2015/12/rex-app-development-and-migration.html).
The 2017.1 version of the REX SDK contains new sample project `DRevitFreezeDrawing` that was created from a discontinued REX module.
Many thanks to Paweł Madej, Senior Software Engineer in Krakow, Poland, for pointing it out and supplying the following detailed description:
Subject
Separate a drawing or view from model so that the state of the drawing or view stays unchanged (frozen).
Summary
This sample contains the discontinued Revit Extensions module Freeze Drawing.
It enables you to separate a drawing or view from an object model so that its state remains unchanged (frozen) or export it to a DWG file.
REXSDK Values
- Real module with complete UI and help.
- Fully functional REX module.
- Presents sample source code for view exporting and importing.
Description
The extension is based on the Revit DWG Import and DWG Export functionality.
All frozen drawings or views are placed in newly created views.
Selected views are imported to DWG files with user-defined parameters.
Freezing of drawings does not include 3D views or sheets.
The extension also offers options for saving frozen drawings and views on the disk.
Instructions
- Copy the module to the `...Modules/Category` directory.
- Open Revit.
- Define model.
- Run the extension.
- Choose the view to be exported or frozen.
- Press OK to see result and new Drafting View (Detail).
- From now on, the frozen view will remain even if the model is changed.
Screen Shots
After compilation of the sample modules, the new command is visible at the bottom of the samples list:
![Freeze Drawing command](img/freeze_drawing_1.png)
The main dialogue prompts whether to freeze the current view or selected ones:
![Freeze Drawing main dialogue](img/freeze_drawing_2.png)
The view selection form looks like this:
![Freeze Drawing view selection form](img/freeze_drawing_3.png)
Thanks again to Paweł for pointing it out!
#### Determining the Height of a Column
There is no API query or property to retrieve the length of a column directly, because it depends on the column type and its relationships with other objects.
Here are two alternative approaches suggested by Jim Jia.
This method uses the bottom and top level parameters to calculate the length of a vertical column:
```csharp
///
/// Determine the height of a vertical column from
/// its top and bottom level.
/// summary>
public Double GetColumHeightFromLevels(
Element e )
{
if( !IsColumn( e ) )
{
throw new ArgumentException(
"Expected a column argument." );
}
Document doc = e.Document;
double height = 0;
if( e != null )
{
// Get top level of the column
Parameter topLevel = e.get_Parameter(
BuiltInParameter.FAMILY_TOP_LEVEL_PARAM );
ElementId ip = topLevel.AsElementId();
Level top = doc.GetElement( ip ) as Level;
double t_value = top.ProjectElevation;
// Get base level of the column
Parameter BotLevel = e.get_Parameter(
BuiltInParameter.FAMILY_BASE_LEVEL_PARAM );
ElementId bip = BotLevel.AsElementId();
Level bot = doc.GetElement( bip ) as Level;
double b_value = bot.ProjectElevation;
// At this point, there are a number of
// additional Z offsets that may also affect
// the result.
height = ( t_value - b_value );
}
return height;
}
```
In this approach, a number of Z offset parameters on the column instance may affect the final result.
The second method retrieves the column bounding box and uses its maximum and minimum coordinates to determine its height:
```csharp
///
/// Determine the height of any given element
/// from its bounding box.
/// summary>
public Double GetElementHeightFromBoundingBox(
Element e )
{
// No need to retrieve the full element geometry.
// Even if there were, there would be no need to
// compute references, because they will not be
// used anyway!
//GeometryElement ge = e.get_Geometry(
// new Options() {
// ComputeReferences = true } );
//
//BoundingBoxXYZ boundingBox = ge.GetBoundingBox();
BoundingBoxXYZ bb = e.get_BoundingBox( null );
if( null == bb )
{
throw new ArgumentException(
"Expected Element 'e' to have a valid bounding box." );
}
return bb.Max.Z - bb.Min.Z;
}
```
This approach is more generic and can be used for any geometric BIM element.
Furthermore, the result obtained from the bounding box already takes the column Z offset parameters into account.
More approaches are possible, e.g., some columns have a location curve, from which its start and end points can be determined.
You can also query the start and end points of the column's analytical line. Be aware that this may not be in the same location as the physical element.
Thanks to Jim for sharing these snippets!
I also added them
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2017.0.131.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.131.1) in the module [CmdColumnRound.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdColumnRound.cs#L30-L104), cf. the [diff from the previous release](https://github.com/jeremytammik/the_building_coder_samples/compare/2017.0.131.0...2017.0.131.1).