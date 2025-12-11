---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: qa
optimization_date: '2025-12-11T11:44:14.014839'
original_url: https://thebuildingcoder.typepad.com/blog/0477_content_loc_dwg_setting.html
post_number: '0477'
reading_time_minutes: 2
series: general
slug: content_loc_dwg_setting
source_file: 0477_content_loc_dwg_setting.htm
tags:
- family
- revit-api
title: Reading DWG Settings and Content Location from INI File
word_count: 483
---

### Reading DWG Settings and Content Location from INI File

Autumn is here in full now, grey and wet outside.
The beautiful yellow leaves are turning brown and soggy.
We had a wonderful sunny weekend, though, and I lay on the grass in the sunshine watching the distant mountains yesterday...

In a completely different vein, today is the final day for submitting the Autodesk University material!
I'm still not completely done...

Meanwhile, here are two quick cases handled by Joe Ye quite a while back which both ended up resorting to reading data from the Revit INI file:

- [DWG settings](#1).- [Content location](#2).

#### DWG Settings

**Question:** I am exporting a Revit model to DWG and IFC format.
Before exporting, I need to fetch the settings option.
Is it possible to fetch the Revit DWG and IFC export settings option thru API?

**Answer:** Revit doesnt expose any API to retrieve the DWG or IFC export setting options.
However, the current option file name being used is saved in Revit.ini file.
For example, here are the setting option filenames stored in my version of RAC 2011:

```
ExportLayersNameDWG=..\..\..\..
  \ProgramData\Autodesk\RAC 2011
  \exportlayers-dwg-CP83.txt

ExportToClassIFC=..\..\..\..
  \ProgramData\Autodesk\RAC 2011
  \exportlayers-ifc-IAI.txt

ExportLayersNameDGN=..\..\..\..
  \ProgramData\Autodesk\RAC 2011
  \exportlayers-dgn-BS1192.txt
```

As the names indicate, ExportLayersNameDWG is for DWG, ExportToClassIFC for IFC, and ExportLayersNameDGN for the DGN export settings.

The Revit.ini file is always located in the same folder as the executable Revit.exe itself.
If Revit was installed using the default path settings, this folder is C:\Program Files\Autodesk\Revit Architecture 2011\Program.

You can read the settings using the standard .NET file content reading API.

#### Content Location

**Question:** I am wondering if there are is way for me to find out where the content (families etc.) is placed on computers with any Revit 2011 version?

When I looked for the installation location of the Revit application, there were six different registry keys that I could look for, depending on the version (Architecture,
Structure, MEP) and 32/64 bit.

I am interested in installing my families to the correct location, in the same place as the other content.

**Answer:** The family library path can be retrieved from the Revit.ini configuration file in each products Program folder.
Here are the settings providing the family library path (they are all in one single long line in the Revit.ini file):

##### RAC2011

```
DataLibraryLocations
  =Imperial Library=C:\ProgramData
    \Autodesk\RAC 2011\Imperial Library
  ,Imperial Detail Library=C:\ProgramData
    \Autodesk\RAC 2011\Imperial Library
    \Detail Components
```

##### RST2011

```
DataLibraryLocations
  =Imperial Library=C:\ProgramData
    \Autodesk\RST 2011\Imperial Library
  ,Imperial Detail Library=C:\ProgramData
    \Autodesk\RST 2011\Imperial Library
    \Detail Components
```

Retrieving the family library path from the Revit.ini configuration file should be independent of the Revit flavour and the 32 or 64 bit OS version.

Many thanks to Joe for these suggestions.