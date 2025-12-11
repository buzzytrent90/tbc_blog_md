---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.2
content_type: qa
optimization_date: '2025-12-11T11:44:17.056768'
original_url: https://thebuildingcoder.typepad.com/blog/1925_2022_1_bld_inst_dev.html
post_number: '1925'
reading_time_minutes: 9
series: general
slug: 2022_1_bld_inst_dev
source_file: 1925_2022_1_bld_inst_dev.md
tags:
- csharp
- parameters
- revit-api
- rooms
- sheets
- walls
- windows
title: 2022 1 Bld Inst Dev
word_count: 1886
---

### Revit 2022.1 SDK, RevitLookup Build and Install
Exciting news with a lot of changes to RevitLookup and The Building Coder samples:
- [Revit 2022.1 SDK released](#2)
- [`WallCrossSection` vs. `WallCrossSectionDefinition`](#3)
- [RevitLookup build and install](#4)
- [Bye-bye lookup builds](#5)
- [The Building Coder samples revamped](#6)
- [Copy as HTML update](#7)
- [Image cleanup and a robot arm](#8)
#### Revit 2022.1 SDK Released
The Revit 2022.1 SDK has been released in
the [Revit Developer Center](https://www.autodesk.com/developer-network/platform-technologies/revit).
The initial Revit 2022 SDK dates from April 12, 2021.
The updated Revit 2022.1 SDK was released on October 28, 2021.
It includes important enhancements addressing new Revit product functionality and developer wishes and requests.
A list of what's new is provided in the SDK documentation in the \*What's New\* section and the \*Changes and Additions\* document.
I'll put that information online for easier searching and finding asap.
#### WallCrossSection vs. WallCrossSectionDefinition
The Revit 2022.1 unfortunately introduced a breaking change:
`WallCrossSection` was renamed to `WallCrossSectionDefinition`
The Revit team is working on a knowledge base article to document this and provide a recommendation on how to handle it.
The issue was brought to our attention by [@ricaun](https://github.com/ricaun), Luiz Henrique Cassettari, in
his [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [RevitApi 2022 update change WallCrossSection to WallCrossSectionDefinition](https://forums.autodesk.com/t5/revit-api-forum/revitapi-2022-update-change-wallcrosssection-to/td-p/10720345):
\*\*Question:\*\* I was messing with `GroupTypeId` and found something interesting and a little worrying.
On my computer, I have the Revit 2022 first release.
- Product Version: 20210224_1515(x64)
- RevitApi.dll Version: 22.0.2.392
My plugin application uses
the NuGet package [Revit_All_Main_Versions_API_x64](https://www.nuget.org/packages/Revit_All_Main_Versions_API_x64),
and I usually use the last version:
- Product Version: 20210921_1515(x64)
- RevitApi.dll Version: 22.1.1.516
The `GroupTypeId` property `WallCrossSection` changed to `WallCrossSectionDefinition` in the Revit 2022.1 update.
If I use the old property, it will throw an exception on the new version.
I use the new one, the first Revit 2022 release will break.
I will probably never use this `ForgeTypeId`, but I worry that other things could change in this update.
Should I be worried about my application breaking each time a new Hotfix is released?
\*\*Answer:\*\* Very sorry about this mishap!
No, you should not need to worry about that; this is an exceptional case and an error in the update release.
Thank you very much for pointing it out!
This launched a significant discussion in the development team.
First of all, it did indeed happen as you noted.
Secondly, it should not have happened.
They are discussing both how to handle this specific issue and how to prevent anything similar from occurring in the future.
The simple workaround for the moment is to use the integer value of the enum to cover both 2022.0 and 2022.1.
They discussed reverting back again in future updates, but that would cause even more disruption.
They discussed defining both enumerations with the same underlying integer values.
For the moment, just using the underlying integers is the safest way to go, I guess:
```csharp
// BuiltInParameterGroup.PG_WALL_CROSS_SECTION_DEFINITION; // -5000228,
// BuiltInParameterGroup.PG_WALL_CROSS_SECTION; // -5000228,
var PG_WALL_CROSS_SECTION = (BuiltInParameterGroup) (-5000228);
```
#### RevitLookup Build and Install
Moving on to more positive news, we were blessed last week with a very exciting contribution to create
a [modeless version of RevitLookup](https://thebuildingcoder.typepad.com/blog/2021/10/bridges-regeneration-and-modeless-revitlookup.html).
This rapid evolution continues with an untiring stint of contributions
from Roman [@Nice3point](https://github.com/Nice3point) and
his extensive series of pull requests:
- [100](https://github.com/jeremytammik/RevitLookup/pull/100) – Fix naming and implement pattern matching
- [101](https://github.com/jeremytammik/RevitLookup/pull/101) – Cleanup and build system
- [102](https://github.com/jeremytammik/RevitLookup/pull/102) – Changelog and remove unused files
- [104](https://github.com/jeremytammik/RevitLookup/pull/104) – Fix snoop db exception
- [105](https://github.com/jeremytammik/RevitLookup/pull/105) – Update badges
- [107](https://github.com/jeremytammik/RevitLookup/pull/107) – Renaming
- [108](https://github.com/jeremytammik/RevitLookup/pull/108) – Multi-version installer
- [110](https://github.com/jeremytammik/RevitLookup/pull/110) – Update Readme.md
As a result, RevitLookup now boasts a modern up-to-date build system, a multi-version installer, a separate GitHub developer branch `dev`, and many other enhancements:
![RevitLookup installer](img/revitlookup_installer.png "RevitLookup installer")
Here are some of Roman's explanations from our pull request conversations:
- Corrected the style of the code in accordance with the latest guidelines. Access modifiers and some unused variables and methods are not affected. The .sln file has been moved to the root folder, otherwise the development environment will not capture the installer project and other supporting files.
- In recent commits, I have integrated the build system from my template https://github.com/Nice3point/RevitTemplates. Now the installer will build directly to github. After installation, I launched Revit, everything seems to work. For debugging added copying to AppData\Roaming\Autodesk\Revit\Addins\2022. To local build, net core 5 is required. If you still have version 3, please update
- You can check how git action works here https://github.com/Nice3point/RevitLookup/actions/runs/1392275582 Artifacts will have an installer
- to build installer on local machine install this via terminal dotnet tool install Nuke.GlobalTool --global Then you can run nuke command. Watch the video https://github.com/Nice3point/RevitTemplates/wiki/Installer-creation
- moved the changelog to a separate file.
- I think you can use this to automatically create a release https://github.com/marketplace/actions/create-release or https://github.com/marketplace/actions/automatic-releases or https://github.com/marketplace/actions/github-upload-release-artifacts
- Removed the version numbers from the .csproj file. This is redundant information and removes duplication. This solution is not a nuget package and is not used in other projects. Therefore, the most correct decision is to change the installer version number, the dll version number in this case does not affect anything. For now, all you need to remember is to update the version number here https://github.com/jeremytammik/RevitLookup/blob/master/installer/Installer.cs#L19 and generate a new guid https://github.com/jeremytammik/RevitLookup/blob/master/installer/Installer.cs#L37 after Revit 2023 is released
- Now the version of the file will be the same for the installer and dll. Will only be listed in the RevitLookup.csproj file
- The `20` prefix in `2022` can be left to keep the usual versioning. Here's what I found about the version limitation in Windows http://msdn.microsoft.com/en-us/library/aa370859%28v=vs.85%29.aspx
- Added all the latest versions of the plugin to the installer. DLLs are stored in the "Releases" folder, if you think that some versions are too outdated, delete the folder from there, the build system picks up the builds automatically, no hardcode. Adding the current assembly, now it is 2022, is not necessary, will cause a conflict. The current assembly is added after the project is compiled
- Kept the old documentation somewhere, as history, for nostalgic reasons, in honour of Jim Awe. It is authored by Jim Awe, the original implementor of both RevitLookup and the corresponding AutoCAD snooping tool, so it has historical value in itself, i think.
- you don't need to run nuke to debug. Only the green arrow on the VisualStudio panel. Nuke is used only for the purpose of building a project, it simplifies building if, for example, the project has several configurations, for example, for the 20th, 21st and 22nd versions of revit, Nuke build all dll variants at once. The build system is only needed to release a product.
- Also, the project was refactored taking into account the latest versions of the C# language.
Some places have been optimized; for the ribbon, I created extension methods to enable shortening of lengthy repetitive data like this:
```csharp
optionsBtn.AddPushButton(
new PushButtonData(" HelloWorld "," Hello World ... ",
ExecutingAssemblyPath, typeof (HelloWorld) .FullName));
```
It can now be written like this:
```csharp
optionsBtn.AddPushButton(typeof (HelloWorld),
" HelloWorld "," Hello World ... ");
```
The latest versions of the C# language allow you to write like this:
```csharp
MApp.DocumentClosed += m_app_DocumentClosed;
```
instead of
```csharp
m_app.DocumentClosed
+= new EventHandler(
m_app_DocumentClosed);
```
Ever so many thanks to Roman for all his inspired work moving this tool forward and especially his untiring efforts supporting me getting to grips with the new technology!
#### Bye-Bye Lookup Builds
Until now, you could download the most recent build of RevitLookup
from [lookupbuilds.com](https://lookupbuilds.com),
provided by [Build Informed](https://www.buildinformed.com),
[implemented back in 2017](https://thebuildingcoder.typepad.com/blog/2017/04/forgefader-ui-lookup-builds-purge-and-room-instances.html#3) and
diligently maintained ever since by [Peter Hirn](https://github.com/peterhirn).
The new build system broke that and replaces it, as discussed in
the [issue #103 – Gitlab pipeline broken](https://github.com/jeremytammik/RevitLookup/issues/103).
Ever so many thanks once again to Peter for all his work setting up and supporting lookupbuilds over the past years!
#### The Building Coder Samples Revamped
As if all the above were not enough, Roman also went ahead and
revamped [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples),
submitting [pull request #18 – Refactoring](https://github.com/jeremytammik/the_building_coder_samples/pull/18):
> Same as for RevitLookup.
I did not add a build system, there is only one command in the project that displays a small dialog box.
Switched msbuild to Net SDK, now the .csproj file contains much fewer lines.
Added automatic copying of dll and addin to Revit addins folder.
Some variables cannot be renamed as it causes conflicts, manual renaming will take too long, left as is.
There are some unoptimized sections left in the project, but their fixing requires manual labour, there are a lot of files, so I left it as it is.
Changelog has been moved to a separate file.
Mainly this is the addition of new features of the latest versions of C#, refactoring according to the latest guidelines.
Integrated and cursorily tested
in [release 2022.1.152.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2022.1.152.2).
Many thanks again to Roman for this!
Now I just have to figure out how to get back my preferred personal C# editor and formatting settings...
#### Copy as HTML Update
With the revamped version of The Building Coder Samples, I also finally moved from Visual Studio 2017 to 2019 to compile it.
I hope I am the last of the Revit add-in developer community to do so.
It lacked the `Copy as HTML` option that I use for C# source code colourisation, so I revisited my 2019 note
on [updating Copy as HTML](https://thebuildingcoder.typepad.com/blog/2019/09/face-intersect-face-is-unbounded.html#5) and
again installed
the [Productivity Power Tools 2017/2019](https://marketplace.visualstudio.com/items?itemName=VisualStudioPlatformTeam.ProductivityPowerPack2017) via
Extensions > Manage Extensions.
#### Image Cleanup and a Robot Arm
Finally, two nice little snippets that caught my attention, a useful image editor tool and impressive novel hardware technology:
- [Quick and free alternative for cleaning up artifacts in images](https://cleanup.pictures)
- [Robotic arm with full range of motion and static strength](https://youtu.be/guDIwspRGJ8)