---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.6
content_type: qa
optimization_date: '2025-12-11T11:44:13.515574'
original_url: https://thebuildingcoder.typepad.com/blog/0189_porting_to_vb.html
post_number: 0189
reading_time_minutes: 4
series: general
slug: porting_to_vb
source_file: 0189_porting_to_vb.htm
tags:
- csharp
- references
- revit-api
- vbnet
- views
- windows
title: Porting from C# to VB.NET
word_count: 885
---

### Porting from C# to VB.NET

Yesterday, we mentioned Kean Walmsley's recent post on
[converting between C# and VB.NET](http://through-the-interface.typepad.com/through_the_interface/2009/07/converting-between-c-and-vbnet.html).
In response to the closely related frequently asked question on porting the Revit SDK samples provided in C# to VB.NET, here is an exhaustive study of the topic by Adam Nagy.

**Question:**
I'm looking at the Revit SDK samples, but many of them are in C#.
Since I am a VB.NET programmer I'd prefer to use a VB.NET version.
I tried to convert them myself but ran into some problems.
Could you tell me the exact steps I could follow?

**Answer:**
The best thing is to show it through an example.
Let's port the ImportExport sample to VB.NET.
The
[attached sample](zip/importexport-vb.net.zip)
only contains the DWG Import/Export part.

1. Open the "ImportExport" C# project.- Open another instance of Visual Studio.- In the second instance, create a VB.NET "Class Library" project with "Name" = "VB.NET", "Location" = "[Revit SDK]\Samples\ImportExport" and "Create directory for solution" unticked.
       This way, all the project content will automatically be placed in the "[Revit SDK]\Samples\ImportExport\VB.NET" folder instead of e.g. "[Revit SDK]\Samples\ImportExport\VB.NET\ImportExport" folder.- Select the "VB.NET" project in the "Solution Explorer" and then click on it again so you can rename it to "ImportExport".- Select the Solution in the "Solution Explorer" (top node) and in the "Properties" window change its "(Name)" to "ImportExport".- Go to Project > Properties > Application and set "Assembly Name" to "ImportExport" and "Root namespace" to "".- Right-click on the project in the "Solution Explorer" and select "Add reference..."; on the "Browse" tab, navigate to "[Revit install folder]\Program\RevitAPI.dll" and click "OK".- Click the "Show All Files" button in the top of the "Solution Explorer", then select "References > RevitAPI.dll" and in the "Properties" window set "Copy Local" to "False".- Select "Class1.vb" and in the "Properties" window set "File Name" to "Command.vb".
                   A dialog might pop up asking if you want to perform a rename in the project - click "Yes".- Copy all the content of Command.cs - just select Command.cs in the C# project, select its content (Ctrl+A), and copy it (Ctrl+C).- Now convert the code to VB.NET by using e.g. one of the online tools -
                       <http://www.developerfusion.com/tools/convert/csharp-to-vb>
                       seemed to work fine.
                       Just paste (Ctrl+V) the code into the edit window on the site - if it already has some content in it, then first select all of that (Ctrl+A).
                       Then click "Convert to VB.NET" and once the conversion is finished click "copy to clipboard".- Select all the contents of Command.vb (Ctrl+A) and replace it with the converted code, which has been copied to the clipboard (Ctrl+V).
                         In case of interface implementations in VB.NET you need to explicitly define what interface function a given function implements - in C# it's not needed.
                         So when you copied over the content of "Command.cs" to "Command.vb" then you need to insert "Implements IExternalCommand.Execute" at the end of the line of "Public Function Execute(...".- You have to go through all the other files and repeat the above steps.- To be consistent with the samples' structure either.
                             1. replace "Namespace Revit.SDK.Samples.ImportExport.CS" with "Namespace Revit.SDK.Samples.ImportExport.VB.Net" in all "\*.vb" files,- or delete "Namespace Revit.SDK.Samples.ImportExport.CS" in all "\*.vb" files (including Designer files) and set "Project > Properties > Application > Root namespace" to "Revit.SDK.Samples.ImportExport.VB.Net".- Now you can just copy "[Revit SDK]\Samples\ImportExport\CS\Revit.ini" to "C:\Revit SDK\Samples\ImportExport\VB.NET" and replace "Revit.SDK.Samples.ImportExport.CS.Command" with "Revit.SDK.Samples.ImportExport.VB.Net.Command" in it.

In case of simple code files, e.g. "MainData.cs" you just need to create a similar file

1. Right-click the VB.NET project in the "Solution Explorer" and select "Add > Class... > Common Items > Class" and set "Name" to "MainData.vb".- Copy all the content from "MainData.cs" and then paste its converted version into "MainData.vb" as we've done above with "Command.cs".

In case of form files, e.g. "MainForm.cs"

1. Right-click the VB.NET project in the "Solution Explorer" and select "Add > Windows Form..." and set "Name" to "MainForm.vb".- Since we earlier clicked the "Show All Files" button in the top of the "Solution Explorer", we can see that there is also a "Designer" file listed under "MainForm.vb" in the "Solution Explorer".- First copy the converted code of "MainForm.Designer.cs" to "MainForm.Designer.vb" - this will automatically copy all the controls and their event handlers over as well.- Right-click on "MainForm.cs", select "View Code", and copy the converted code to "MainForm.vb".

To retain the same project structure as in the original ImportExport sample, you can simply add folders to the VB.NET project:
Right-click the project and select "Add > New Folder".
You can also drag & drop files inside the project to reorganize them.

Thank you very much Adam for this detailed description!