---
post_number: "1995"
title: "Lookup"
slug: "lookup"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'parameters', 'references', 'revit-api', 'rooms', 'sheets', 'views', 'windows']
source_file: "1995_lookup.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1995_lookup.html"
---

### 64 Bit Ids, Revit and RevitLookup Updates
Important updates to both Revit and RevitLookup, and other interesting news:
- [Revit X.Y.2 updates](#2)
- [RevitLookup 2024.0.5](#3)
- [Backward compatible 64 bit element id](#4)
- [15-minute cities](#5)
- [Cloud data carbon footprint](#6)
- [Live annotated `https` request log](#7)
#### Revit X.Y.2 Updates
Update number 2 has been released for both Revit 2023 and 2024, Revit 2023.1.2 and Revit 2024.0.2, respectively:
![Revit X.Y.2 update](img/2023-05-15_rvt_updates.png "Revit X.Y.2 update")
As usual, they can be obtained from [manage.autodesk.com](http://Manage.Autodesk.com).
#### RevitLookup 2024.0.5
[RevitLookup](https://github.com/jeremytammik/RevitLookup) also sports a new update,
[release 2024.0.5](https://github.com/jeremytammik/RevitLookup/releases/tag/2024.0.5).
Here is a list of all updates and their enhancements since the initial 2024 release:
- [RevitLookup 2024.0.5](https://github.com/jeremytammik/RevitLookup/releases/edit/2024.0.5):
- Static members support: RevitLookup now supports the display of these and other properties and methods:

```
  Category.GetCategory();
  Document.GetDocumentVersion()
  UIDocument.GetRevitUIFamilyLoadOptions()
  Application.MinimumThickness
```

![Snoop static members](img/revitlookup_static_members.png "Snoop static members")
- Ribbon update: SplitButton replaced by PullDownButton.
Thanks for [voting](https://github.com/jeremytammik/RevitLookup/discussions/159)!
![SplitButton menu](img/revitlookup_splitbutton.png "SplitButton menu")
- Other improvements:
- Added DefinitionGroup support
- Added Element.GetMaterialArea support
- Added Element.GetMaterialVolume support
- Added FamilyInstance.get_Room support
- Added FamilyInstance.get_ToRoom support
- Added FamilyInstance.get_FromRoom support
- "Show element" is no longer available for ElementType
- Bugs:
- Fixed issue when GetMaterialIds method didn't return nonPaint materials
[#163](https://github.com/jeremytammik/RevitLookup/issues/163)
- [RevitLookup 2024.0.4](https://github.com/jeremytammik/RevitLookup/releases/edit/2024.0.4):
- Improvements:
- Added Workset support
- Added WorksetTable support
- Added Document.GetUnusedElements support
- Bugs:
- Fixed Dashboard window startup location
- [RevitLookup 2024.0.2](https://github.com/jeremytammik/RevitLookup/releases/edit/2024.0.2):
- Bugs:
- Fixed Fatal Error on Windows 10 https://github.com/jeremytammik/RevitLookup/issues/153
Accent colour sync with OS now only available in Windows 11 and above. Many thanks to [Aleksey Negus](https://t.me/a_negus) for testing builds
- [RevitLookup 2024.0.1](https://github.com/jeremytammik/RevitLookup/releases/edit/2024.0.1):
- Major changes:
- Added option to enable hardware acceleration (experimental)
- Added button to enable RevitLookup panel on Modify tab by @ricaun in [#152](https://github.com/jeremytammik/RevitLookup/pull/152)
Disabled by default. Thanks vor [voting](https://github.com/jeremytammik/RevitLookup/discussions/151)!
- Opening RevitLookup window only when the Revit runtime context is available https://github.com/jeremytammik/RevitLookup/issues/155
- Improvements:
- Added shortcuts support for the Modify tab https://github.com/jeremytammik/RevitLookup/issues/150
- Added EvaluatedParameter support
- Added Category.get_Visible support
- Added Category.get_AllowsVisibilityControl support
- Added Category.GetLineWeight support
- Added Category.GetLinePatternId support
- Added Category.GetElements extension
- Added Reference.ConvertToStableRepresentation support
- Bugs:
- Fixed rare crashes in EventMonitor on large models
- Fixed Curve.Evaluate resolver using EndParameter as values
- Other:
- Added installers for previous RevitLookup versions https://github.com/jeremytammik/RevitLookup/wiki/Versions
Ever so many thanks to Luiz Henrique [@ricaun](https://github.com/ricaun) Cassettari and above all
to Roman [Nice3point](https://github.com/Nice3point) for this impressive list of enhancements, with extra kudos to Roman for all the RevitLookup maintenance work!
#### Backward Compatible 64 Bit Element Id
Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas
shares code for handling `ElementId` 64 bit backward compatibility in the thread
on [upgrade 2024 API causing schema error](https://forums.autodesk.com/t5/revit-api-forum/upgrade-2024-api-causing-schema-error/td-p/11953147),
explaining:
> The change to `Int64` should be transparent for most situations;
there is a design decision some developers will need to consider in terms of how the `ElementId` `IntegerValue` property is replaced with the old `Value`.
I decided it was better to update backwards the base code with an extension method `ElementId.Value`.
Can't do much about the constructor, however:

```
Module RT_ElementIdExtensionModule

# If RvtVer >= 2024 Then

    Public Function NewElementId(L As Long) As ElementId
        Return New ElementId(L)
    End Function
# Else

    Public Function Value(ID As ElementId) As Long
        Return ID.IntegerValue
    End Function

    Public Function NewElementId(L As Long) As ElementId
        If L > Int32.MaxValue OrElse L < Int32.MinValue Then
            Throw New OverflowException("Value for ElementId out of range.")
        End If
        Return New ElementId(CInt(L))
    End Function
# End If

End Module
```

Many thanks to Richard for sharing this approach.
#### 15-Minute Cities
I just noticed Kean Walmsley's nice discussion
of [15-minute cities, 20-minute neighbourhoods and 30-second offices](https://www.keanw.com/2023/02/15-minute-cities-20-minute-neighbourhoods-and-30-second-offices.html).
#### Cloud Data Carbon Footprint
You have probably seen hundreds if not thousands of email footers reminding you not to print every email you receive on paper.
Now the time has come to add a reminder not to save ever bit of information you receive digitally either.
Apparently, storing 100 GB of data in the cloud results in a carbon footprint of about 0.2 tons of CO2 per year.
That is about the same as:
- Burning 45 kg of coal
- Driving a car for approximately 965 km
- The production of about 1,000 plastic bags
#### Live Annotated Https Request Log
Talking about bits stored in the cloud, here is a neat web page that enables you
to [see this page fetch itself, byte by byte, over TLS](https://subtls.pages.dev/):
> This page performs a live, annotated `https:` request for its own source.