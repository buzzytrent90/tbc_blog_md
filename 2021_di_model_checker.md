---
post_number: "2021"
title: "Di Model Checker"
slug: "di_model_checker"
author: "Jeremy Tammik"
tags: ['csharp', 'python', 'revit-api', 'sheets', 'views', 'windows']
source_file: "2021_di_model_checker.md"
original_url: "https://thebuildingcoder.typepad.com/blog/2021_di_model_checker.html"
---

### Dependency Injection and Model Checker API
Happy New Year!
Let's begin it gently with the following notes on topics that caught my eye and interest:
- [AU 2023 classes](#2)
- [Dependency injection for Revit API](#3)
- [RevitLookup updates](#4)
- [Model checker API docs](#5)
- [ChatGPT and Maestro AI for Revit scripting](#6)
- [Construction spending rising in the US](#7)
- [Free Will](#8)
- [Vuca](#9)
#### AU 2023 Classes
Did you miss an interesting class at AU?
Check out the entire collection
of [Autodesk University 2023 classes online](https://www.autodesk.com/autodesk-university/search?fields.year=2023).
#### Dependency Injection for Revit API
Between Christmas and New Year,
Luiz Henrique [@ricaun](https://ricaun.com/) Cassettari implemented, documented and shared a complete solution
for [dependency injection for Revit API](https://forums.autodesk.com/t5/revit-api-forum/dependency-injection-for-revit-api/td-p/12467760),
saying:
> I created a library to help create a container for Dependency Injection, designed to work with Revit API.
It is open-source and has a package in the Nuget:
> - [github.com/ricaun-io/ricaun.Revit.DI](https://github.com/ricaun-io/ricaun.Revit.DI)
- [www.nuget.org/packages/ricaun.Revit.DI](https://www.nuget.org/packages/ricaun.Revit.DI)
> I created this [22-minute video](https://youtu.be/Q_greabHlUQ) on how to add the package and a simple example with an `ICommand` implementation:
> - [github.com/ricaun-io/RevitAddin.DI.Example](https://github.com/ricaun-io/RevitAddin.DI.Example)

> That's it for the year 2023; Happy New Year with best regards!
Happy New Year to you too, *ricaun*, and to the entire community from me as well!
#### RevitLookup Updates
Before the DI project, ricaun also contributed to RevitLookup, working with
Roman [Nice3point](https://github.com/Nice3point), principal maintainer, helping to produce:
- [RevitLookup release 2024.0.11](https://github.com/jeremytammik/RevitLookup/releases/tag/2024.0.11)
- [RevitLookup release 2024.0.12](https://github.com/jeremytammik/RevitLookup/releases/tag/2024.0.12)
RevitLookup 2024.0.11 welcomes you with improved visuals, support for templates to fine-tune data display, improved navigation, in-depth color support:
- Navigation. Updated navigation allows `Ctrl + Click` in the tree or grid to open any selected item or group of items in a new tab.
This also allows you to analyze items that RevitLookup doesn't support, e.g., looking at StackTrace for exceptions.
- Color Preview. Changes to the user interface give us the ability to customize the display of any type of data.
Now you will be able to visually see how materials or ribbon look.
`Autodesk.Revit.DB.Color` and `System.Windows.Media.Color` are supported.
- Update available notification\*\*. Updates are now checked automatically and an icon is displayed in the navigation area if a new version is available.
- Background effects. Available on windows 11 only: Acrylic, Blur; the visual representation of the background depends on your desktop image and current theme.
- Color extensions. Convert color to other formats HEX, CMYK, etc. Color name identification, `en` and `ru` localizations available. `Autodesk.Revit.DB.Color` and `System.Windows.Media.Color` are supported.
- Fixed incorrect display when switching themes on Windows 10.
- Returned deleted notification when checking for updates.
- Updated developer's [guide](https://github.com/jeremytammik/RevitLookup/blob/dev/Contributing.md#styles).
- Full changelog: [2024.0.10...2024.0.11](https://github.com/jeremytammik/RevitLookup/compare/2024.0.10...2024.0.11)
Here, I'm wrapping things up. Wishing everyone a splendid New Year and a joyous Christmas ahead. As always, yours truly – @Nice3point
RevitLookup 2024.0.12 is the last corrective update for 2023, bringing minor tweaks and improvements:
- Add theme update for all open RevitLookup instances by @ricaun in [#200](https://github.com/jeremytammik/RevitLookup/pull/200)
- Fix incorrect Hue calculation for some colour formats
- Disable all background effects for Windows 10. Thanks @ricaun for help and testing [#194](https://github.com/jeremytammik/RevitLookup/issues/194)
- Full changelog: [2024.0.11...2024.0.12](https://github.com/jeremytammik/RevitLookup/compare/2024.0.11...2024.0.12)
That's all for now.
Again, wishing you all a Happy New Year with best regards, do what you love, evolve, travel, don't forget to have a rest and keep coding! – @ricaun
#### Model Checker API Docs
*Shrey_shahE5SN4* very kindly points out
the [Model Checker API documentation](https://help.autodesk.com/view/AIT4RVT/ENU/?guid=InteroperabilityToolsForRevit_040mcxr_0404mcxr_html) in
his question
on [setting up `IPreBuiltOptionsService` options for CheckSet in AIT](https://forums.autodesk.com/t5/revit-api-forum/setting-up-iprebuiltoptionsservice-options-for-checkset-in-ait/td-p/12455815):
> I am... building an add-in button.
When clicked, it will execute the Model Checker from Autodesk Interoperability Tools.
Following the provided guidelines, I am progressing through the necessary steps:
> [Model Checker API](https://help.autodesk.com/view/AIT4RVT/ENU/?guid=InteroperabilityToolsForRevit_040mcxr_0404mcxr_html)
Thank you for that hint, Shrey_shahE5SN4.
#### ChatGPT and Maestro AI for Revit Scripting
AI programming assistants are boosting developer effectivity in many areas.
Here is one dedicated to Revit customisation:
[Maestro AI for Revit scripting](https://maestro.bltsmrt.com/).
Looking forward to hearing how it shapes out.
Eric Boehlke of [Truevis](https://truevis.com) has also been working to focus LLMs to work better with programming Revit and shared some results:
> My latest attempt:
> - [OpenAI bim-coding-coach](https://chat.openai.com/g/g-7gcy5wueV-bim-coding-coach)
- [BIM-Coding-Coach GitHub repository](https://github.com/truevis/BIM-Coding-Coach)
> I haven't tested it with C# yet, but it is working well with Python and DesignScript.
#### Construction Spending Rising in the US
Good news for the AEC industry: an impressive positive jump
in [total construction spending: manufacturing in the United States (TLMFGCONS)](https://fred.stlouisfed.org/series/TLMFGCONS#0):
![Total construction spending](img/total_construction_us.png "Total construction spending")
#### Free Will
As a scientifically and technically minded person, I often find philosophical pondering rather vague.
I was therefore pleased to read the interesting and precise analytical philosophical discussion
on [Dennett vs Sapolsky on free will: a clash over different claims?](https://philosophy.stackexchange.com/questions/106926/dennett-vs-sapolsky-on-free-will-a-clash-over-different-claims),
comparing the volition and predetermination of a boulder crashing down a mountain and a skier who skis down the mountain, including the possible influence of quantum mechanical effects.
#### Vuca
Have you ever heard the term "vuca"?
I had not.
Apparently, it stands for volatility, uncertainty, complexity and ambiguity.
Facing us in the recent past, and possibly in coming years as well.
Which leads to the dread of a long-term state, a “permavucalution”.
Oh dear.
Let's hope that our humanity and free will can help handle it.