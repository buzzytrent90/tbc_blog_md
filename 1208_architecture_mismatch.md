---
post_number: "1208"
title: "Architecture Mismatch Warning Disabler Update"
slug: "architecture_mismatch"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api']
source_file: "1208_architecture_mismatch.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1208_architecture_mismatch.html"
---

### Architecture Mismatch Warning Disabler Update

The default Visual Studio settings will generate a
[processor architecture mismatch warning](http://thebuildingcoder.typepad.com/blog/2013/06/processor-architecture-mismatch-warning.html) when
compiling a Revit 2014 or 2015 add-in.

Last year, I implemented a utility named DisableMismatchWarning.exe to
[recursively disable this warning](http://thebuildingcoder.typepad.com/blog/2013/07/recursively-disable-architecture-mismatch-warning.html) in
all projects in all subfolders of the current directory.

Now I just happened to receive a new sample add-in project from a developer for testing that was not recognised by this utility, so I implemented an
[update](#3) for it.

Before getting to that, let me share some pictures from our full moon fire last night by my friend Anja:

![The Moon](file:////j/photo/jeremy/2014/2014-09-08_tuellinger_full_moon/2014-09-08_20.27.26_moon.jpg)

The Moon

![The Fire](file:////j/photo/jeremy/2014/2014-09-08_tuellinger_full_moon/2014-09-08_20.46.46_fire.jpg)

The Fire

![The Moon Cloud Bed](file:////j/photo/jeremy/2014/2014-09-08_tuellinger_full_moon/2014-09-08_20.26.14_moon_cloud_bed.jpg)

The Moon Cloud Bed

#### DisableMismatchWarning Update

To process the new Visual Studio project file I received, I simply added support for additional Import tags in the first few lines.

This should work fine for both C# and VB projects.

The updated version is available from the
[DisableMismatchWarning GitHub repository](https://github.com/jeremytammik/DisableMismatchWarning).