---
post_number: "0159"
title: "Export Family Instance to gbXML"
slug: "export_instance_to_gbxml"
author: "Jeremy Tammik"
tags: ['doors', 'family', 'references', 'revit-api', 'transactions', 'walls', 'windows']
source_file: "0159_export_instance_to_gbxml.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0159_export_instance_to_gbxml.html"
---

### Export Family Instance to gbXML

The
[Green Building XML Schema](http://www.gbxml.org)
or gbXML was developed to facilitate the transfer of building information stored in CAD building information models, enabling interoperability between building design models and a wide variety of engineering analysis tools.
It has been widely adopted by leading CAD vendors and is on the road to becoming a de-facto industry standard schema, so we obviously see more and more application developers interested in using and supporting it, leading to questions such as the following one.

**Question:**
When exporting a Revit MEP model to gbXML, only system families, doors and windows are exported.
All other families such as Generic, etc. are missing in the gbXML output.
This includes almost all the families that we use as shading devices, which are generic families.
How can we resolve this?

**Answer:**
If you launch the export via the API, you can specify certain settings before calling the Export method.
Please set ProjectInfo.gbXMLSettings.ExportComplexity to the enumeration value defining the degree of complexity you wish to export.
Here are the possible settings, which are documented in the Revit API help file RevitAPI.chm:

- Simple: Curtain Walls and Curtain Systems are exported with one huge opening with the total opening area equal to all openings; a curtain wall with 50 panels is exported as 1 opening.- SimpleWithShadingSurfaces: Export with simple type and shading surfaces.- Complex: Curtain Walls and Curtain Systems are exported with several openings, panel by panel; a curtain wall with 50 panels is exported as 50 openings.- ComplexWithShadingSurfaces: Include complex type and shading surfaces.- ComplexWithMullionsAndShadingSurfaces: Include complex type, shading surfaces and mullion. Mullions in Curtain Walls and Systems are exported as shading surfaces. A "simplified" analytical shading surface is produced from a mullion based on its centerline, thickness and offset.

Unfortunately, none of these settings actually cause the generic family instances to be included in the export.

After testing all ExportComplexity cases via the user interface settings for Export Complexity, I see that none of them export generic family instances.

One option would be to model the devices using system families instead, since the component families are not exported.
For example, using a small slab might be a possibility.

Thanks to Joe Ye for handling this case.

### Two typical Problems

I just handled two cases with typical recurring problems that I would like to remind everybody to watch out for:

- [Use modal dialogues](#1).- [Set Copy Local to False](#2).

#### Use modal dialogues

You should always use modal dialogues in your Revit external command, unless you really know what you are doing.
Otherwise, you may quickly run into problems:

**Question:**
I am trying to load a type symbol and am having issues with it.
The method returns true indicating that the type symbol has been loaded, but it does not show up anywhere in the Revit user interface.
For instance, I am loading a window type from a family that has never been loaded before.
I would expect to see it in the Revit user interface afterwards, however it is not there.
Right now we have to cancel the family browser and reset the project browser for the family or type to load.
I am pretty sure that we are getting our wires crossed.

**Answer:**
I looked at your sample application, and yes, I can tell you what the problem is.
Crossed wires is actually a pretty apt description.
This is your application execution sequence:

- The command is launched.- The dialogue box is displayed. It is modeless, so it remains visible forever or until explicitly closed.- The command terminates and returns IExternalCommand.Result.Succeeded.- The user interacts with the dialogue, for instance by pushing the 'Begin Test' button.

Unfortunately, in the last step, you are no longer in a context in which the Revit API will allow you to interact with it in any way, because your command has already terminated.

Please use a modal dialogue instead, and do not return from the command until your use of the Revit API is concluded. When you return, the transaction that Revit manages for your command is terminated. Your return code of Succeeded, Failed or Cancelled determines whether the transaction is committed or aborted.

Some hints on modal versus modeless dialogues are discussed in the posts on
[driving Revit from outside](http://thebuildingcoder.typepad.com/blog/2008/12/driving-revit-from-outside.html)
and
[using modeless dialogues](http://thebuildingcoder.typepad.com/blog/2009/02/revit-window-handle-and-modeless-dialogues.html).
More information on the automatic transaction handling performed by Revit for your external command is discussed in the introduction to
[transactions](http://thebuildingcoder.typepad.com/blog/2009/01/transactions.html),
[transaction responsibility](http://thebuildingcoder.typepad.com/blog/2009/01/transaction-responsibility.html)
and
[the document IsModified property](http://thebuildingcoder.typepad.com/blog/2008/12/document-ismodified-property.html).

#### Set Copy Local to False

Always ensure that the Copy Local flag on your application's reference to the RevitAPI.dll assembly in your Visual Studio project is set to false, as described in the post on
[the document IsModified property](http://thebuildingcoder.typepad.com/blog/2008/09/the-sdk-samples.html).
I still keep seeing cases where this flag is left in its default setting of True.