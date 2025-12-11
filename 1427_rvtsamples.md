---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.4
content_type: tutorial
optimization_date: '2025-12-11T11:44:15.994215'
original_url: https://thebuildingcoder.typepad.com/blog/1427_rvtsamples.html
post_number: '1427'
reading_time_minutes: 8
series: general
slug: rvtsamples
source_file: 1427_rvtsamples.md
tags:
- family
- python
- revit-api
- rooms
- sheets
- vbnet
- windows
title: Rvtsamples
word_count: 1693
---

### RvtSamples for Revit 2017
Prompted by
Dan Tartaglia [commented](http://thebuildingcoder.typepad.com/blog/2016/04/revit-2017-revitlookup-and-sdk-samples.html#comment-2633447580) on
the [Revit 2017 SDK sample compilation](http://thebuildingcoder.typepad.com/blog/2016/04/revit-2017-revitlookup-and-sdk-samples.html), saying:
> A couple of other issues to note about the SDK examples (RvtSamples).
>
> 1. The Icons folder has to be copied to where the RvtSamples.dll exists (probably obvious to most).
> 2. RvtSamples.txt: there are only 6 statements for PlacementOptions.dll (the description is missing).
> 3. RvtSamples.txt: The assembly path for CreateShared.dll is not correct (for both paths).
That was the first time in history that someone else took a look at RvtSamples for a new major release of Revit before I did.
Yipee!
So I guess I'd better get going with it as well:
- [Setting up RvtSamples for Revit 2017](#2)
- [Copy Html Markup in Visual Studio 2015](#3)
- [Running Revit 2017 in the Visual Studio 2015 debugger](#4)
- ['Security – Unsigned Add-In' message](#5)
- [RvtSamples DLL and TXT should be together](#6)
- [Specifying the Revit SDK samples root path](#7)
- [Correcting errors in individual SDK sample entries](#8)
- [PlacementOptions description line is missing](#9)
- [The five FabricationPartLayout external commands](#10)
- [RvtSamples loads and RvtSamples.txt is cleaned up](#11)
#### Setting up RvtSamples for Revit 2017
The first thing I did was add the RvtSamples add-in manifest and input text file to its project files:
![RvtSamples add-in manifest and input text file](img/RvtSamples_addin_manifest_input_txt.png)
Then in `Application.cs`, I set the Boolean test variable `testClassName` to true:
```csharp
bool testClassName = true; // jeremy
```
Search for 'jeremy' to find it.
Now RvtSamples will complain on any external command specified in `RvtSamples.txt` that cannot be found.
#### Copy Html Markup in Visual Studio 2015
While writing this, I notice that I have to install something like `CopySourceAsHtml` in my Visual Studio 2015 IDE before I can start describing my steps here.
First things first.
It was not completely trivial
to [enable CopySourceAsHtml for Visual Studio 2012](http://thebuildingcoder.typepad.com/blog/2014/04/migrating-roomeditorapp-to-revit-2015.html#5).
I tried to use a similar procedure for Visual Studio 2015, e.g.:

```
> md "C:\Users\tammikj\Documents\Visual Studio 2015\Addins"

> copy "C:\Users\tammikj\Documents\Visual Studio 2012\Addins" "C:\Users\tammikj\Documents\Visual Studio 2015\Addins"
C:\Users\tammikj\Documents\Visual Studio 2012\Addins\CopySourceAsHtml.AddIn
  1 file(s) copied.
```

I edited the add-in file analogously to the last installation, only to discover – after some significant effort and frustration –
[Visual Studio 2015: goodbye add-ins, hello VsPackage extensions](http://visualstudioadventures.com/2015/04/25/visual-studio-2015-goodbye-add-ins-hello-vspackage-extensions).
So I will have to set up an entirely new system for this now.
After quite a bit of fruitless searching I ended up installing
the [Productivity Power Tools 2015](https://visualstudiogallery.msdn.microsoft.com/34ebc6a2-2777-421d-8914-e29c1dfa7f5d).
Now I once again have my – or 'a' – 'Copy Html Markup' menu entry:
![Copy Html Markup menu entry](img/visual_studio_2015_copy_html_markup.png)
It does a really good job!
This is the RvtSamples add-in manifest, retrieved via the new tool, pasted into the blog post text completely unmodified:

```
</spanxml version="1.0" encoding="utf-8"?>
<RevitAddIns>
  <AddIn Type="Application">
    <Name>External ToolName>
    <Assembly>C:\a\lib\revit\2017\SDK\Samples\RvtSamples\CS\bin\Debug\RvtSamples.dllAssembly>
    <ClientId>42cb0a70-2ee7-4e64-a42d-87b9cdcc41c8ClientId>
    <FullClassName>RvtSamples.ApplicationFullClassName>
    <VendorId>ADSKVendorId>
    <VendorDescription>Autodesk, www.autodesk.comVendorDescription>
  AddIn>
RevitAddIns>
```

Now, to continue with RvtSamples...
#### Running Revit 2017 in the Visual Studio 2015 Debugger
Next, I set up the RvtSamples project to launch Revit 2017 in the Visual Studio debugger.
Starting up Revit 2017 in the Visual Studio 2015 debugger does not work at all initially:
![Debugging Revit 2017 in Visual Studio 2015](img/revit_2017_debug_01.png)
Happily, this issue has been addressed, e.g. in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread discussing
[cannot start Revit 2015 for API debugging](http://forums.autodesk.com/t5/revit-api/cannot-start-revit-2015-for-api-debugging/td-p/4978060) and
[Visual Studio 2015](http://forums.autodesk.com/t5/revit-api/visual-studio-2015/td-p/5647387).
They suggest to "Use Managed Compatibility Mode", found under Tools > Options > Debugging, and enabling native code debugging, respectively.
I opted for the former:
![Use Managed Compatibility Mode](img/revit_2017_debug_02.png)
With that setting, I am now able to start up Revit 2017 in the Visual Studio 2015 debugger and continue with my RvtSamples installation.
#### 'Security – Unsigned Add-In' Message
Revit asked me to sign in using my Autodesk account.
It then brought up the new security dialogue, explaining that the RvtSamples add-in assembly that I am attempting to load is not equipped with a trusted signature, and asking me to confirm loading it:
![Security – Unsigned Add-In message](img/revit_2017_rvt_samples_02.png)
In text format, this reads:

```
[Window Title]
Security - Unsigned Add-In

[Main Instruction]
The publisher of this add-in could not be verified. What do you want to do?

[Content]
Name:       External Tool
Publisher:  Unknown Publisher
Location:   C:\a\lib\revit\2017\SDK\Samples\RvtSamples\CS\bin\Debug\RvtSamples.dll
Issuer:     None
Date:       2016-04-20 15:28:45

Make sure that this add-in comes from a trusted source.

[Always Load] [Load Once] [Do Not Load]

[Footer]
What are the risks?
```

I selected 'Always Load' for this one.
#### RvtSamples DLL and TXT Should be Together
Once RvtSamples itself is properly loaded, it reads its input text file `RvtSamples.txt`, which specifies the locations of all the external commands to add to the RvtSamples toolbox.
On my first attempt, the RvtSamples external application is loaded and is unable to locate its input text file:
![RvtSamples.txt not found](img/revit_2017_rvt_samples_03.png)
We need to explicitly set up the path to it, or, alternatively and simpler, modify the RvtSamples.dll output location to the RvtSamples `CS` root folder, so they are both in the same directory; then it will be found automatically.
Obviously, the add-in manifest needs to be updated to that path as well, then.
#### Specifying the Revit SDK Samples Root Path
With the Boolean test variable `testClassName` set to true, as described above, every entry in the text file is verified.
Since I have not updated the paths specified in the input text file yet, the very first entry generates an error:
![Invalid path specified in RvtSamples.txt](img/revit_2017_rvt_samples_04.png)
So let's edit those paths.
In RvtSamples.txt, I replaced all occurrences of `C:\Revit Copernicus SDK\Samples\` by my SDK samples root folder `C:\a\lib\revit\2017\SDK\Samples\`.
187 occurrences were replaced.
#### Correcting Errors in Individual SDK Sample Entries
Restart debugging.
This time, we get further.
The VB.NET sample DeleteObject generates an error in line 63.
![DeleteObject VB.NET sample error](img/revit_2017_rvt_samples_05.png)
That one, and several other VB.NET samples as well, have always caused some strange issues for me, as I already [pointed out in previous releases](http://thebuildingcoder.typepad.com/blog/2013/04/compiling-the-revit-2014-sdk.html#9).
I ignored that one, and other VB.NET issues as well.
#### PlacementOptions Description Line is Missing
The issue that Dan pointed out about a missing description for the PlacementOptions sample is reported rather cryptically like this:
![Missing description for the PlacementOptions sample](img/revit_2017_rvt_samples_06.png)
I added the description line and restarted the debugger.
#### The Five FabricationPartLayout External Commands
Skipping a few more VB.NET errors, the FabricationPartLayout sample apparently has a wrong external command name:
![Invalid external command name for the FabricationPartLayout sample](img/revit_2017_rvt_samples_07.png)
Not only that, but it also has its `CopyLocal` flag set to `true` on the Revit API assemblies, thus creating local copies of them all.
This is not recommended, quite the contrary, so I filed an issue REVIT-90156 [API SDK Sample: FabricationPartLayout Revit API assemblies copy local is true] with the development team to have it fixed.
I also noticed another important omission.
The FabricationPartLayout sample defines five different external commands:
- ConvertToFabrication
- FabricationPartLayout
- OptimizeStraights
- RenumberingPart
- StretchAndFit

```
C:\...\FabricationPartLayout\CS > grep class.*IExtern *.cs
ConvertToFabrication.cs:   public class ConvertToFabrication : IExtern...
FabricationPartLayout.cs:   public class FabricationPartLayout : IExte...
OptimizeStraights.cs:   public class OptimizeStraights : IExternalCommand
RenumberingPart.cs:   public class RenumberingPart : IExternalCommand
StretchAndFit.cs:   public class StretchAndFit : IExternalCommand
```

All five should be added to RvtSamples.txt. e.g. like this:

```
MEP
FabricationPartLayout: ConvertToFabrication
Fabrication Part sample layout
LargeImage:
Image:
C:\...\FabricationPartLayout\CS\bin\Debug\FabricationPartLayout.dll
Revit.SDK.Samples.FabricationPartLayout.CS.ConvertToFabrication

MEP
FabricationPartLayout: FabricationPartLayout
Fabrication Part sample layout
LargeImage:
Image:
C:\...\FabricationPartLayout\CS\bin\Debug\FabricationPartLayout.dll
Revit.SDK.Samples.FabricationPartLayout.CS.FabricationPartLayout

MEP
FabricationPartLayout: OptimizeStraights
Fabrication Part sample layout
LargeImage:
Image:
C:\...\FabricationPartLayout\CS\bin\Debug\FabricationPartLayout.dll
Revit.SDK.Samples.FabricationPartLayout.CS.OptimizeStraights

MEP
FabricationPartLayout: RenumberingPart
Fabrication Part sample layout
LargeImage:
Image:
C:\...\FabricationPartLayout\CS\bin\Debug\FabricationPartLayout.dll
Revit.SDK.Samples.FabricationPartLayout.CS.RenumberingPart

MEP
FabricationPartLayout: StretchAndFit
Fabrication Part sample layout
LargeImage:
Image:
C:\...\FabricationPartLayout\CS\bin\Debug\FabricationPartLayout.dll
Revit.SDK.Samples.FabricationPartLayout.CS.StretchAndFit
```

I submitted another issue for this, REVIT-90157 [API SDK samples: RvtSamples lists wrong external command for FabricationPartLayout].
Next, the CreateShared DLL is reported as missing:
![CreateShared missing](img/revit_2017_rvt_samples_08.png)
That is another one of those pesky VB.NET samples, so I'll ignore it.
#### RvtSamples Loads and RvtSamples.txt is Cleaned Up
That was it!
RvtSamples loads, and RvtSamples.txt has been cleaned up a bit:
![RvtSamples in Revit 2017](img/revit_2017_rvt_samples_09.png)
Here is my first final version of [RvtSamples.txt](zip/RvtSamples_2017.txt) for Revit 2017.
If anyone feels like telling us how to fix the VB.NET sample errors, please be my guest.
Oh, and I must not forget to turn off the `testClassName` flag again, or it will slow down loading tremendously and pop up all those warning messages each time I restart Revit:
```csharp
bool testClassName = false; // jeremy
```
I hope this will save some effort on your part.
Good luck and have fun with the Revit 2017 API!
Lots more to come on this subject!