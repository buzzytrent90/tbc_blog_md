---
post_number: "2055"
title: "Lookup"
slug: "lookup"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'references', 'revit-api', 'sheets']
source_file: "2055_lookup.md"
original_url: "https://thebuildingcoder.typepad.com/blog/2055_lookup.html"
---

### RevitLookup 2025.0.9
A new version of RevitLookup is available, and AI influence observable:
- [RevitLookup 2025.0.9](#2)
- [Revit.ini file editor](#2.1)
- [Dependency conflict static analyzer](#2.2)
- [Public RevitLookup roadmap](#2.3)
- [Other improvements](#2.4)
- [RevitLookup 2025.0.10](#2.5)
- [LLMs influence academic language](#3)
#### RevitLookup 2025.0.9
Roman [@Nice3point](https://t.me/nice3point) Karpovich, aka Роман Карпович,
published [RevitLookup](https://github.com/jeremytammik/RevitLookup)
[release 2025.0.9](https://github.com/jeremytammik/RevitLookup/releases/tag/2025.0.9)
with important enhancements by himself,
[RichardPinka](https://github.com/RichardPinka) and [SergeyNefyodov](https://github.com/SergeyNefyodov):
- [Revit.ini file editor](#2.1)
- [Dependency conflict static analyzer](#2.2)
- [Public RevitLookup roadmap](#2.3)
#### Revit.ini File Editor
The \*\*Revit.ini\*\* file is a key configuration file in Revit that stores settings related to user preferences, system behavior, and project defaults.
The \*\*Revit.ini File Editor\*\* provides a simple and efficient way to manage these settings without the need for manual editing.
With this tool, users can quickly adjust Revit’s configurations to match project needs or personal preferences, making it an essential utility for both professionals and teams working with Revit.
![Revit.ini file editor](img/revitlookup_2025_0_9_1.png "Revit.ini file editor")
This is our first public version, and we are excited for you to try it out for yourself!
Make sure to file issues you encounter on our GitHub so we can continue to improve it.
For more details, please refer to the:
- [Revit.ini file editor documentation](https://github.com/jeremytammik/RevitLookup/wiki/Revit.ini-File-Editor)
#### Dependency Conflict Static Analyzer
Some users experience issues launching RevitLookup, often caused by conflicts with third-party plugins, cf., [issue 269](https://github.com/jeremytammik/RevitLookup/issues/269).
To help resolve these issues, we introduced new dependency reporting tools that allow you to analyze, identify and upgrade problematic plugins causing crashes:
![Dependency conflict static analyzer](img/revitlookup_2025_0_9_2.png "Dependency conflict static analyzer")
- [Download DependenciesReport tool](https://github.com/jeremytammik/RevitLookup/issues/269#issuecomment-2323309590)
Many thanks to @RichardPinka for testing tools in the discussion of [issue 281](https://github.com/jeremytammik/RevitLookup/issues/281).
#### Public RevitLookup Roadmap
Curious about what’s next?
Keep up to date on the latest developments for RevitLookup and share your feedback.
Check out the [Public RevitLookup Roadmap](https://github.com/users/jeremytammik/projects/1) to see what’s coming up in future releases.
![Public RevitLookup roadmap](img/revitlookup_2025_0_9_3.png "Public RevitLookup roadmap")
#### Other Improvements
\*\*New extensions\*\* by @SergeyNefyodov; type, name and short description:
- Pipe – [HasOpenConnector](https://github.com/jeremytammik/RevitLookup/pull/261) – Checks if there is open piping connector for the pipe
- Family – [FamilyCanConvertToFaceHostBased](https://github.com/jeremytammik/RevitLookup/pull/263) – Indicates whether the family can be converted to face host based
- Family – [GetProfileSymbols](https://github.com/jeremytammik/RevitLookup/pull/263) – Gets the profile Family Symbols
- Document – [GetLightFamily](https://github.com/jeremytammik/RevitLookup/pull/266) – Creates a light family object from the family document
- LightFamily – [GetLightTypeName](https://github.com/jeremytammik/RevitLookup/pull/266) – Return the name for the light type
- LightFamily – [GetLightType](https://github.com/jeremytammik/RevitLookup/pull/266) – Return a LightType object for the light type
- Application – [GetMacroManager](https://github.com/jeremytammik/RevitLookup/pull/268) – Gets the Macro manager from the application
- Document – [GetMacroManager](https://github.com/jeremytammik/RevitLookup/pull/268) – Gets the Macro manager from the document
\*\*New API support:\*\*
- [\*\*CylindricalFace\*\* class support](https://github.com/jeremytammik/RevitLookup/issues/264):
- Radius property support
- [\*\*StructuralSettings\*\* class support](https://github.com/jeremytammik/RevitLookup/pull/282) by @SergeyNefyodov:
- GetStructuralSettings method support
- [\*\*StructuralSettings\*\* class support](https://github.com/jeremytammik/RevitLookup/pull/283) by @SergeyNefyodov:
- GetActiveSunAndShadowSettings method support
- GetSunrise method support
- GetSunset method support
- GetSunset method support
- IsTimeIntervalValid method support
- IsAfterStartDateAndTime method support
- IsBeforeEndDateAndTime method support
- [\*\*RevisionNumberingSequence\*\* class support](https://github.com/jeremytammik/RevitLookup/pull/289) by @SergeyNefyodov:
- GetAllRevisionNumberingSequences method support
- [\*\*AnalyticalLinkType\*\* class support](https://github.com/jeremytammik/RevitLookup/pull/288) by @SergeyNefyodov:
- IsValidAnalyticalFixityState method support
- [\*\*AreaVolumeSettings\*\* class support](https://github.com/jeremytammik/RevitLookup/pull/287) by @SergeyNefyodov:
- GetAreaVolumeSettings method support
- GetSpatialElementBoundaryLocation method support
\*\*New default settings:\*\*
- `Show Static` members enabled by default
- `Show Events` enabled by default
- `Show Extensions` enabled by default
\*\*Bugs:\*\*
- [Fixed missing quick access icon](https://github.com/jeremytammik/RevitLookup/issues/267)
- [Fixed DataGrid accent color](https://github.com/jeremytammik/RevitLookup/issues/273)
\*\*Miscellaneous:\*\*
- Updated \*\*Contributing\*\* guide
- Added new GitHub \*\*issue templates\*\*
- [Full changelog](https://github.com/jeremytammik/RevitLookup/compare/2025.0.8...2025.0.9)
- [RevitLookup versioning](https://github.com/jeremytammik/RevitLookup/wiki/Versions)
#### RevitLookup 2025.0.10
Shortly after, Roman went on to publish [RevitLookup 2025.0.10](https://github.com/jeremytammik/RevitLookup/releases/tag/2025.0.10):
- [Fixed placeholder for the dark theme](https://github.com/jeremytammik/RevitLookup/issues/291)
- Fixed the Revit.ini editor filter button name
- Fixed the Revit.ini editor filter placeholder
- [Disabled the `Visual.Enter` method](https://github.com/jeremytammik/RevitLookup/issues/292)
- [Suppressed `GenericHost` startup messages](https://github.com/jeremytammik/RevitLookup/pull/294) by @Nefarion
- [Full changelog](https://github.com/jeremytammik/RevitLookup/compare/2025.0.9...2025.0.10)
- [RevitLookup versioning](https://github.com/jeremytammik/RevitLookup/wiki/Versions)
#### LLMs Influence Academic Language
In a different vein, looking at AI-related news, according to scientific studies, LLMs are already influencing human academic language,
cf., [The Impact of Large Language Models in Academia: from Writing to Speaking](https://www.alphaxiv.org/abs/2409.13686).
Less scientifically rigorous, similar results are also raised in the El Pais article stating
that [excessive use of words like ‘commendable’ and ‘meticulous’ suggests ChatGPT has been used in thousands of scientific studies](https://english.elpais.com/science-tech/2024-04-25/excessive-use-of-words-like-commendable-and-meticulous-suggest-chatgpt-has-been-used-in-thousands-of-scientific-studies.html).