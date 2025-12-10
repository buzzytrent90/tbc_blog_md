---
post_number: "1126"
title: "RevitLookup for Revit 2015"
slug: "revitlookup_2015"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'references', 'revit-api', 'sheets', 'views']
source_file: "1126_revitlookup_2015.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1126_revitlookup_2015.html"
---

### RevitLookup for Revit 2015

My first post dealing with Revit 2015 is dedicated to RevitLookup, the most important Revit database exploration tool, both for developers and interested non-developers.

This is particularly urgent, since RevitLookup no longer is included in the standard Revit SDK (software development kit).

It is now available from the
RevitLookup GitHub repository instead.

I created a preliminary version of RevitLookup for the Revit 2015 Meridian pre-release, just to ensure that everybody who needs access to this tool has it available right away.

![RevitLookup in Revit 2015](img/revitlookup_2015.png)

It compiles and runs perfectly fine, although some compilation warnings on use of deprecated API functionality are displayed.

It currently refers to the Revit API assemblies located in the Revit Meridian root installation folder.
That path needs to be updated to compile for an official release of Revit 2015.

#### Migration

Here are the steps I performed for the migration, which was extremely straightforward:

1. Replaced the RevitAPI.dll and RevitAPIUI.dll references.
2. Changed the .NET framework from 4.0 to 4.5.
3. Rebuilt all. It compiles successfully, generating
   [0 errors and 24 warnings](zip/revit_lookup_2015_warnings_01.txt).
4. Fixed a few of the deprecated API usage occurrences:

- Rewrote FamilyUtil.GetFamilySymbol using GetFamilySymbolIds instead of Symbols.
- Ditto in TypeSelectorForm.GetAvailableSymbols.
- Ditto in Importer.UpdateFamilySymbol.
- Rewrote TestElements.ViewToNewSheetHardwired to use ViewSheet.GetAllPlacedViews instead of ViewSheet.Views.

5. Updated the version number to 2015.0.0.0.

The fixes are currently marked with a comment:

```
  // jeremy migrated from Revit 2014 to 2015:
```

That will probably be removed again soon, since the same information can be easily gleaned from the version control system.

#### Download

For the complete source code, Visual Studio solution and add-in manifest, please refer to the
[RevitLookup GitHub repository](https://github.com/jeremytammik/RevitLookup).

The version discussed above is stored there as
[release 2015.0.0.0](https://github.com/jeremytammik/RevitLookup/releases/tag/2015.0.0.0).

It compiles successfully, currently still generating
[0 errors and 19 warnings](zip/revit_lookup_2015_warnings_02.txt).