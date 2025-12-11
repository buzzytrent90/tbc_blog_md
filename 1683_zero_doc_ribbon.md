---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:16.516178'
original_url: https://thebuildingcoder.typepad.com/blog/1683_zero_doc_ribbon.html
post_number: '1683'
reading_time_minutes: 6
series: general
slug: zero_doc_ribbon
source_file: 1683_zero_doc_ribbon.md
tags:
- elements
- references
- revit-api
- sheets
- views
title: Zero Doc Ribbon
word_count: 1194
---

### Support DLL, Zero Doc State and Forge Accelerators
If you are interested in diving deep into Forge programming, don't miss the upcoming deadline for proposals for the Boston Forge accelerator!
Looking at the Revit API, I implemented a sample demonstrating how
to [enable ribbon items in zero document state](http://thebuildingcoder.typepad.com/blog/2011/02/enable-ribbon-items-in-zero-document-state.html) back
in 2011, for Revit 2011.
How long would it take to port such an add-in to Revit 2019?
Well, this one took ten or twenty minutes, including some debugging, screen snapshots, and documentation.
Admittedly, it is an absolutely trivial example :-)
Let's look at that, and another recurring topic, on loading add-in support DLLs:
- [Migrating the ZeroDocPanel to Revit 2019](#2)
- [Loading add-in support DLLs](#3)
- [Rome and Boston Forge accelerators](#4)
![External command in zero document state](img/zero_doc_button_2019.png)
#### Migrating the ZeroDocPanel to Revit 2019
John raised an issue in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`IsCommandAvailable` causing a nullref exception](https://forums.autodesk.com/t5/revit-api-forum/iscommandavailable-causes-nullref-exception/m-p/8258750).
That prompted me to update my original implementation
to [enable ribbon items in zero document state](http://thebuildingcoder.typepad.com/blog/2011/02/enable-ribbon-items-in-zero-document-state.html) in
Revit 2011 and test it in Revit 2019.
I performed the flat migration from Revit 2011 to Revit 2019, i.e., updated the Revit API assembly references and set the .NET framework to version 4.7.
Next, I tried to load and debug the add-in.
Revit displayed an error due to a lack of the `VendorId` tag in the add-in manifest:
> External Tools – No VendorId Node for Add-in
> Failed to initialize the add-in “ZeroDocPanel.addin” because the add-in registration is missing the required value of VendorId node. The VendorId node identify the vendor of the add-in application. For Revit to run the add-in, you must register the node defined in manifest “ZeroDocPanel.addin” file.
![VendorId node missing in add-in manifest](img/no_vendorid_node.png)
The `VendorId` node had not yet been invented in 2011 :-)
I added the vendor id, and all was well.
I debugged the add-in and ensured that the `OnStartup` and `IsCommandAvailable` methods are both called just as expected.
The external command is listed and can be executed in the external tools menu in zero document state, cf. the screen snapshot above.
The command execution displays its little task dialogue:
![Zero document state external command displaying a task dialogue](img/zero_doc_cmd_msg_2019.png)
The complete add-in sample including Visual Studio project, source code and add-in manifest now lives in its
own [ZeroDocPanel GitHub repository](https://github.com/jeremytammik/ZeroDocPanel).
Here are [all the commits I made for the migration](https://github.com/jeremytammik/ZeroDocPanel/commits/master):
- Initial commit from original blog post
- Added author and license
- Flat migration to Revit 2019 – updated Revit API assembly references and .NET framework to version 4.7
- Removed references to Revit 2011
- Added `VendorId` node
- Added assembly description
- Added link to original blog post and new question thread
#### Loading Add-In Support DLLs
In
another [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on loading the [missing `System.ComponentModel.Annotations` v4.2.0.0](https://forums.autodesk.com/t5/revit-api-forum/missing-system-componentmodel-annotations-v4-2-0-0/m-p/8255546),
my colleague Jim Jia suggested using an assembly resolver reacting to the `AppDomain.CurrentDomain.AssemblyResolve` event:
> You may try to force loading the assembly if it cannot be loaded successfully. The code snippet below should provide a safe solution, per my experience; hope it is helpful:
```csharp
Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
// Try to load assembly if Revit command fails to load it.
AppDomain.CurrentDomain.AssemblyResolve
+= new ResolveEventHandler(
OnAssemblyResolve );
//
// your other code...
//
}
Assembly OnAssemblyResolve( object sender, ResolveEventArgs args )
{
Assembly a = null;
if( args.Name.Contains( "ComponentModel.Annotations" ) )
{
string assemblyFile = Path.Combine( addinDir,
"ComponentModel.Annotations.dll" );
if( File.Exists( assemblyFile ) )
a = Assembly.LoadFrom( assemblyFile );
}
return a;
}
```
We explored some related issues and had success with this approach in the past, e.g., here:
- [Revit Add-in Unit Testing](http://thebuildingcoder.typepad.com/blog/2013/07/revit-add-in-unit-testing.html)
- [Security, Framing Cross Section Analyser and REX](http://thebuildingcoder.typepad.com/blog/2013/12/security-framing-cross-section-analyser-and-rex.html)
- [RvtVa3c Assembly Resolver](http://thebuildingcoder.typepad.com/blog/2014/05/rvtva3c-assembly-resolver.html)
- [Framing Cross Section Analyser and REX in Revit 2015](http://thebuildingcoder.typepad.com/blog/2015/03/framing-cross-section-analyser-and-rex-in-revit-2015.html)
- [REX Add-In Development and Migration](http://thebuildingcoder.typepad.com/blog/2015/12/rex-app-development-and-migration.html)
- [Add-In Templates Supporting Edit and Continue](http://thebuildingcoder.typepad.com/blog/2017/02/add-in-templates-supporting-edit-and-continue.html)
#### Rome and Boston Forge Accelerators
I recently mentioned that I am attending
the [Forge accelerator in Rome](http://thebuildingcoder.typepad.com/blog/2018/08/revit-20191-cefsharp-forge-accelerator-in-rome.html#5) end
of this month, September 24-28.
I expect that it is too late to decide to join that one now.
However, [you can still submit proposals for the Boston accelerator taking place in October](https://forums.autodesk.com/t5/bim-360-api-forum/forge-accelerator-boston-ma-october-1-5-2018/td-p/8249266)
– note that the deadline for submitting one is next Saturday, September 15:
The Forge team is finishing the review of proposals for the upcoming accelerator taking place in the Autodesk Boston office October 1-5.
Note that the BIM 360 Issue API is now publicly available, cf. the .NET and Node.js
samples [introducing BIM 360 Issues API](https://forge.autodesk.com/blog/introducing-bim-360-issues-api) and
accessing [BIM 360 Issues API with Node.js](https://forge.autodesk.com/blog/bim-360-issues-api-sample-nodejs).
You can now also simulate the use of the [Design Automation API](https://forge.autodesk.com/en/docs/design-automation/v2/developers_guide/overview) for Revit on your machine (note that it is still in private beta!).
Therefore, the Forge team would love to support you and your software development team in working intensively on a Forge-based project making use of these new technologies with one-on-one support and advice from Autodesk Forge API experts.
If you have an idea for a new web or mobile application based on Autodesk Forge APIs, or need help getting an existing application working, then come along to the event.
There is no cost to attend the accelerator, other than your own travel and living expenses.
However, we do ask that you submit a proposal as part of your application so that we can verify that the intended use of the Autodesk Forge APIs in your project is feasible.
You can [apply and submit your proposal here](http://autodeskcloudaccelerator.com/apply).
For more information, please visit the [Forge Accelerator website](http://autodeskcloudaccelerator.com/forge-accelerator).