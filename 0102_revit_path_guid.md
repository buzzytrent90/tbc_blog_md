---
post_number: "0102"
title: "Revit Install Path and Product GUIDs"
slug: "revit_path_guid"
author: "Jeremy Tammik"
tags: ['revit-api', 'windows']
source_file: "0102_revit_path_guid.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0102_revit_path_guid.html"
---

### Revit Install Path and Product GUIDs

Things were a bit hectic with many cases toward the end of last week, so less posts.
By chance and luck, I was able to spontaneously join a ski tour on Sunday.
The danger of avalanches is currently pretty high in most of Switzerland, and the weather was bad with lots of snow as well, so we evaded those problems by going way down south into Ticino and climbing Motto Crostel, 2302 m, from Faido and Tengia at 1099 m.
We even got some sun and spots of deep blue sky in the late afternoon.
Now I am finishing off this post that I started working on a while ago so that I have something prepared for tomorrow morning before I get bogged down in real work again, on the Revit installation path and the Revit product GUIDs.

Installation of a Revit add-in requires knowledge of the Revit program folder path, either just to modify Revit.ini or to actually install the add-in into the same folder.

#### Revit Add-in Folder

The easiest way to make full use of some of the more complex .NET framework features which involve security settings and access rights, for example application settings and resource handling, is to install a class assembly in the same directory as the executable defining the AppDomain loading the it, i.e. Revit.exe in the Revit executable folder, which is the Program subdirectory under the Revit main installation folder.
One example of this is making use of the using System.Configuration namespace to store application settings.
The general recommendation for both AutoCAD.NET and Revit add-ins is for the installer to install the external command into the main application executable folder.

However, it is not absolutely necessary to install a Revit add-in in the same directory as Revit.exe.
I leave all my sample applications in the binary directory where they are built, and some shipping add-ins from Autodesk create their own installation folders, by default under C:/Program Files/Autodesk, such as:

- Batch Print for Revit 2009
- Worksharing Monitor for Autodesk Revit 2010
- STL Export for Autodesk Revit 2010

#### Revit Install Path

In any case, an installer will need to determine the Revit install path.

Revit is designed to write as little as possible into the Windows registry.
Though it adds some entries to the Windows registry at session start-up, by default it will clean up those entries again when it exits.
Fortunately, Windows maintains some independent uninstall information in the registry, and this includes the installation folder path.
The uninstall information is located under the following registry key:

```
HKEY_LOCAL_MACHINE
  \SOFTWARE
  \Microsoft
  \Windows
  \CurrentVersion
  \Uninstall
```

If the 32-bit version of Revit is installed on a 64-bit OS machine, the registry path is placed under the Wow6432Node key, so it becomes:

```
HKEY_LOCAL_MACHINE
  \SOFTWARE
  \Wow6432Node
  \Microsoft
  \Windows
  \CurrentVersion
  \Uninstall
```

For each version of Revit, the installer will add uninstall information under a sub-key named according to a version specific GUID.
These GUIDs depend on the Revit version and flavour.
There are also separate GUIDs for the 32 and 64 bit versions.
The Revit product GUID is thus independent of the Revit language and only depends on the 32 or 64 bit version, major version number and flavour, which is one of RAC, RME or RST, for Revit Architecture, MEP and Structure.

To find the GUID for any specific Revit product version and language that you are interested in, simply search for the product name under the Uninstall key.
For instance, for Revit Architecture 2009, search for the string "Revit Architecture 2009".
The search result will be located in a sub-key named "{A3A37DA6-70C0-497C-BCB1-148E9EC1D32E}", so the uninstall information for an 32 bit English version of Revit Architecture 2009 on a 32-bit OS is located under the following registry key:

```
HKEY_LOCAL_MACHINE
  \SOFTWARE
  \Microsoft
  \Windows
  \CurrentVersion
  \Uninstall
  \{A3A37DA6-70C0-497C-BCB1-148E9EC1D32E}
```

#### Revit Product GUIDs

Here is a set of GUIDs for various versions of Revit, including the upcoming 2010 release:

|  |  |  |  |
| --- | --- | --- | --- |
| 2010 | 64 bit | RAC | {2A8EEE2F-4A9E-43d8-AA07-EC8A316B2DEB} |
| RME | {A1BD042B-8A6F-4e37-92A3-78921BB45B05} |
| RST | {BC9C0A08-DEA4-4138-A7FB-8F68866DB0C1} |
| 32 bit | RAC | {572FBF5D-3BAA-42ff-A468-A54C2C0A17C3} |
| RME | {5C8281B1-B927-495a-A0FF-AB4BDFAE505C} |
| RST | {939D29FC-B82D-42a7-BB1E-8E3F121505CC} |
| 2009 | 64 bit | RAC | {D2466208-7348-4214-B01E-7BC8729E2BD3} |
| RME | {4A98F976-01B5-40e8-A496-AEFD85C3A446} |
| RST | {B354FCF5-CF64-4fa2-AA84-9D9B2A6FA649} |
| 32 bit | RAC | {A3A37DA6-70C0-497C-BCB1-148E9EC1D32E} |
| RME | {E3781DCB-A650-4E66-9B74-67A1B17F052C} |
| RST | {C4B3B3C3-2EE9-48D3-9BF5-4443F7ECF759} |
| 2008 | RAC | {4A11206C-4377-49E8-911E-B11548658FF3} |
| RME | {60A2743E-C881-4880-94ED-96445E38616F} |
| RST | {8D0AE0BB-4FE5-491D-A284-3B363F02E639} |
| 9.0 | Revit Building | {D11DB6CB-0332-4735-B312-B919741D975E} |
| 3 | RST | {3F11CEE0-D30D-41ce-8522-922B5D8BB324} |
| 8.1 | Revit Building | {7EBC0489-5E47-498D-BE31-B094484612E9} |
| 2 | RST | {BE814F63-629D-4fd8-B628-1437AC10F9D4} |