---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: qa
optimization_date: '2025-12-11T11:44:15.174888'
original_url: https://thebuildingcoder.typepad.com/blog/1039_acge_built_in.html
post_number: '1039'
reading_time_minutes: 4
series: general
slug: acge_built_in
source_file: 1039_acge_built_in.htm
tags:
- csharp
- geometry
- levels
- references
- revit-api
- views
title: Using the Built-In Revit AcGe Functionality
word_count: 822
---

### Using the Built-In Revit AcGe Functionality

Anyone who ever used the AutoCAD API for geometrical purposes must be enamoured with AcGe.

On the other hand, the Revit geometry library is pretty meagre in comparison.

Wouldn't it be nice to be able to use all that powerful AcGe functionality in Revit as well?

Actually, if you take a look at the support DLLs included in the Revit main executable directory, AcGe is obviously present:

```
  C:\Program Files\Autodesk\Revit 2014 > dir *acge*

    Directory of C:\Program Files\Autodesk\Revit 2014

  02/05/2013  12:52 PM  1,679,688 acge19.dll
  02/05/2013  12:52 PM    501,064 acgex19.dll

    2 File(s)      2,180,752 bytes
```

Wow.
Wouldn't it be nice to be able to access that?

Well, you can, actually.

Here is a small proof of concept created by Florian Schmid of
[SOFiSTiK AG](http://www.sofistik.com) demonstrating showing the usage of AcGe geometry classes in Revit.

He originally provided it for Revit 2013.
I did not get around to testing and verifying it until now, on Revit 2014, so here are the archive files for both of these versions:

- [acge\_revit\_2013.zip](zip/acge_revit_2013.zip)
- [acge\_revit\_2014.zip](zip/acge_revit_2014.zip)

The latter contains the Visual Studio solution acge\_revit.sln that includes three Visual Studio projects:

- geometry\_acge – a C++/CLI project that links with acge19.lib. The static Converter class provides methods to convert Revit geometry objects to AcGe objects and vice versa. The Helper class provides two higher level methods for curve projection.
- AcGeTest – a C# project implementing an external application that uses the Geometry.AcGe.Helper.orthoProjectIntoPlane method to project some Revit lines into a plane in its OnStartup method.
- AcGeTestCmd – a C# project implementing an external command providing the same functionality in its Execute mainline.

AcGeTest references the geometry\_acge assembly, of course.

The AcGe headers and libs are those from the 2013 ObjectARX or ObjectARX 2014 installations, respectively.
There is no need to modify them in any way.

The acge19.dll is automatically demand loaded when geometry\_acge.dll is loaded.
Because acge19.dll lives in the same location as revit.exe, it is found without problems, so no need to manually load it.

To get the example working, you have to adjust the 'Include Directory' and 'Additional Library Directories' of the geometry\_acge project so that the ObjectARX headers and libraries can be found on your system.

Welcome back AcGe, how we’ve missed you!

Here are the steps I performed to install ObjectARX 2014 and migrate the sample to the new version:

- Download and install
  [ObjectARX](http://usa.autodesk.com/adsk/servlet/item?siteID=123112&id=785550).
  For all further information on that and self-help resources for AutoCAD software-based development efforts, please visit the
  [AutoCAD Developer Center](="http://www.autodesk.com/developautocad"),
  which provides API overviews, getting started presentations, links to API discussion groups and more.
- Update the Revit API assembly references from 2013 to 2014.
- Migrate obsolete code, e.g. calls to NewLineBound, NewLineUnbound and get\_EndPoint were replaced by Line.CreateBound, Line.CreateUnbound and GetEndPoint.
  You might want to take a look at the definition of the Geometry class in the curve\_converter.cpp module of the geometry\_acge project to see examples of how these calls look in C++.
- Fix the
  [architecture mismatch warning](http://thebuildingcoder.typepad.com/blog/2013/07/recursively-disable-architecture-mismatch-warning.html).

The result is an error and warning free compilation (copy and paste somewhere or view source to see truncated lines in full):

```
------ Rebuild All started: Project: geometry_acge, Configuration: Debug x64 ------
  Stdafx.cpp
  AssemblyInfo.cpp
  curve_converter.cpp
  Generating Code...
  .NETFramework,Version=v4.0.AssemblyAttributes.cpp
  geometry_acge.vcxproj -> C:\a\vs\acge_revit\x64\Debug\geometry_acge.dll
------ Rebuild All started: Project: AcGeTest, Configuration: Debug Any CPU ------
  AcGeTest -> C:\a\vs\x64\AcGeTest.dll
========== Rebuild All: 2 succeeded, 0 failed, 0 skipped ==========
```

There was one more hiccup: the debug version had an erroneous C++ compilation setting, generating a Multi-threaded Debug DLL.
I had to change that to generate a Multi-threaded DLL instead, to ensure that the correct runtime library is used.

Many thanks to Florian for pointing out and demonstrating the access to this powerful geometry functionality.

#### Disclaimer

Note that I am not sure of the legality of making use of this undocumented access.

If you plan to use this approach in a commercial product, I suggest checking with the appropriate Autodesk channels first.

I also took this opportunity of adding a
[license](http://thebuildingcoder.typepad.com/blog/about-the-author.html#3) and a
[disclaimer](http://thebuildingcoder.typepad.com/blog/about-the-author.html#4) to
this blog itself, covering all its contents.

I recently discovered the
[importance of a license](http://thebuildingcoder.typepad.com/blog/2013/10/the-building-coder-samples-on-github.html#2) when starting to publish more diligently on GitHub.