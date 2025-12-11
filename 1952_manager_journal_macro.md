---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: news
optimization_date: '2025-12-11T11:44:17.118651'
original_url: https://thebuildingcoder.typepad.com/blog/1952_manager_journal_macro.html
post_number: '1952'
reading_time_minutes: 3
series: general
slug: manager_journal_macro
source_file: 1952_manager_journal_macro.md
tags:
- python
- revit-api
- sheets
- views
title: Manager Journal Macro
word_count: 576
---

### Analysis of Macros, Journals and Add-In Manager
Let's look at the Revit macro study results, add-in manager debug/trace functionality and a Python and Dynamo journal analysis tool:
- [Revit macro study shareback](#2)
- [Add-in manager with debug trace](#3)
- [Journal file analysis](#4)
- [Plugging the HSL colour format](#5)
#### Revit Macro Study Shareback
We recently
asked for feedback from the add-in developer community in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how you use Revit Macros](https://forums.autodesk.com/t5/revit-api-forum/research-how-do-you-use-revit-macros/m-p/11158305).
The results are now in, and we share them with you for further evaluation and feedback:
> Feel free to review the research result summary and add any comments or suggestions at:
> [Revit Macro Study Shareback](https://www.autodeskresearchcommunity.com/hub/posts/post-25914628)
You have to set up an account with Autodesk research, fill in a survey and await the response email to see them.
To save others the same process, time and effort, I took the liberty of printing the results to PDF and sharing them here in [revit_macro_study_shareback.pdf](zip/revit_macro_study_shareback.pdf).
Many thanks to the Revit development team and Siyu Guo for the shareback and interesting results.
#### Add-In Manager with Debug Trace
We recently mentioned
Chuong Ho's [open source add-in manager](https://thebuildingcoder.typepad.com/blog/2022/01/add-in-manager-formulamanager-and-tiger-year.html#2).
> Usually, when developing and debugging an add-in with Revit API, user has to recompile, close and reopen Revit each time they modify the add-in code.
With Add-In Manager, you can modify and run the add-in directly without closing and reopening Revit again and again.
Chuong announces new enhancements:
> Revit Add-in Manager supports Debug/Trace WriteLine including a dockable panel now.
> It's an improvement that I think will save even more debugging time for programmers 🤗
> Download from the [RevitAddInManager GitHub repo](https://github.com/chuongmep/RevitAddInManager).
![Add-in manager with debug trace](img/addinmanager_debugtrace.jpg "Add-in manager with debug trace")
By the way, for the sake of completeness, note that
the [.NET hot reload for editing code at runtime](https://devblogs.microsoft.com/dotnet/introducing-net-hot-reload)
in Visual Studio 2019 also enables you to update your add-in code on the fly, cf.
[apply code changes debugging Revit add-in](https://thebuildingcoder.typepad.com/blog/2021/10/localised-forge-intros-and-apply-code-changes.html#4).
#### Journal File Analysis
I happened to notice
Andreas [@andydandy74](https://github.com/andydandy74) Dieckmann's
interesting Python and Dynamo project [Journalysis](https://github.com/andydandy74/Journalysis).
> Journalysis is a Revit journal, worksharing log and keyboard shortcuts analysis package for the Dynamo visual programming environment.
> Since there is hardly any documentation on Revit journals, it is a slow process.
> I have started writing some documentation in the [wiki](https://github.com/andydandy74/Journalysis/wiki) that may not be entirely complete.
> This package is aimed at automating the analysis of Revit journals and worksharing logs for statistical purposes.
> It helps track and monitor crashes, API errors, memory usage, sync with central duration, keyboard shortcut usage and more.
#### Plugging the HSL Colour Format
Unrelated to the Revit API, I found the 7-minute video explaining and motivating us
to [switch to HSL colour format](https://youtu.be/VInSzHOeFkE) very interesting and informative: