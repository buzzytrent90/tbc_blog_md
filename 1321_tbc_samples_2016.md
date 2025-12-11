---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.766636'
original_url: https://thebuildingcoder.typepad.com/blog/1321_tbc_samples_2016.html
post_number: '1321'
reading_time_minutes: 3
series: general
slug: tbc_samples_2016
source_file: 1321_tbc_samples_2016.htm
tags:
- parameters
- references
- revit-api
- walls
title: Migrating The Building Coder Samples to Revit 2016
word_count: 696
---

### Migrating The Building Coder Samples to Revit 2016

I finally tackled the task of migrating The Building Coder Samples to Revit 2016.

I also have another update on RevitLookup to report:

- [Preparation](#2)
- [Fixing the compilation errors](#3)
- [Installing RvtSamples](#4)
- [RevitLookup update displays all built-in parameter names](#5)

#### Preparation

Before doing anything else, I ensured that the entire project compiles for Revit 2015 with zero warnings.

This guarantees that I am not using any API functionality that was already deprecated in Revit 2015 and therefore removed in 2016.

I then replaced the Revit 2015 API references by the Revit 2016 ones:

![Revit 2016 API references](img/tbc_2016_01_references.png)

After that, compilation produced [6 errors and 30 warnings](zip/bc_migr_2016_a.txt).

#### Fixing the Compilation Errors

The six errors all have the error number CS0246 and are caused by two simple issues:

- The type or namespace name 'ExternalDefinitonCreationOptions' could not be found
- The type or namespace name 'ContFooting' could not be found

The first of these is due to a typo fixed in the Revit 2016 API, a missing 'i' in the spelling of the class name, which is now named [ExternalDefinitionCreationOptions](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html#4.08), as originally intended.

The second missing class is renamed to WallFoundation in Revit 2016, cf. [Structural API changes](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html#4.04) > ContFooting and ContFootingType class and members renamed.

These errors are fixed by renaming ExternalDefinitonCreationOptions to ExternalDefinitionCreationOptions and ContFooting to WallFoundation.
The errors are eliminates and [30 warnings](zip/bc_migr_2016_b.txt) remain.

We completed the first successful compilation of The Building Coder samples for Revit 2016!

I published that right away as [release 2016.0.120.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.120.0).

#### Installing RvtSamples

Before starting to fix the warnings, I thought it might be nice to give it a test run.

To do so, I would like to use the RvtSamples external application add-in to load it.

That is included in the Revit SDK samples and can be compiled together with all the other SDK samples using the Visual Studio solution SDKSamples.sln:

![Revit 2016 SDK samples solution](img/tbc_2016_02_sdk_solution.png)

I downloaded the [Revit SDK installer](http://images.autodesk.com/adsk/files/REVIT_2016_SDK.msi) dated April 23 from the [Revit Developer Centre](http://www.autodesk.com/developrevit) and was able to compile with no problems.

The next step is to set up RvtSamples to load all the SDK samples into Revit.

I edited RvtSamples.txt, replaced the default SDK samples path `C:\Revit 2016 SDK\Samples\` by my specific location, and toggled the `Release` binary directories to `Debug`.

Some paths are still erroneous, e.g.

- C:\a\lib\revit\2016\SDK\Samples\DatumsModification \CS\bin\Debug\DatumsModification.dll

On my system, it needs to be changed to

- C:\a\lib\revit\2016\SDK\Samples\DatumsModification \DatumsModification.dll

With that, RvtSamples loaded successfully and provides access to launch most of the other SDK external command samples:

![RvtSamples in Revit 2016](img/tbc_2016_04_rvtsamples.png)

I added the include directive that I implemented back in 2008 to pull in the BcSamples.txt file to
[load The Building Coder Samples](http://thebuildingcoder.typepad.com/blog/2008/11/loading-the-building-coder-samples.html) to the end of RvtSamples.txt:

```
#include C:\a\lib\revit\2016\bc\BcSamples.txt
```

Lo and behold, now The Building Coder samples are accessible from the RvtSamples collection as well:

![The Building Coder Samples in Revit 2016](img/tbc_2016_05_bcsamples.png)

Happy, happy.

I'll return and fix those pesky warnings some other day.

#### RevitLookup Update Displays All Built-In Parameter Names

Maxence Delannoy [@mdelanno](https://github.com/mdelanno) submitted a new
[pull request #9](https://github.com/jeremytammik/RevitLookup/pull/9) on
RevitLookup to list all the display names associated with a given built-in parameter:

![RevitLookup displays all built-in parameter names](img/revitlookup_all_bip_names.png)

The new version is tagged as
[RevitLookup release 2016.0.0.9](https://github.com/jeremytammik/RevitLookup/releases/tag/2016.0.0.9).

Thank you very much, Maxence!