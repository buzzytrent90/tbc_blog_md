---
post_number: "1530"
title: "Andrey Wizard Debug"
slug: "andrey_wizard_debug"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'sheets']
source_file: "1530_andrey_wizard_debug.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1530_andrey_wizard_debug.html"
---

### Add-In Templates Supporting Edit and Continue
Last week, I presented Andrey
Bushman's [new Visual Studio templates for Revit add-ins](http://thebuildingcoder.typepad.com/blog/2017/02/new-visual-studio-2015-templates-for-revit-add-ins.html).
Furthermore, we discussed many aspects
of [edit and continue](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.49) in
the past, including the solution to use
the [Add-in Manager](http://thebuildingcoder.typepad.com/blog/2016/10/ai-edit-and-continue.html#2).
These two topics have now met and united, because Andrey added support for that and a bunch of other new functionality in
his [commit](https://github.com/Andrey-Bushman/Revit2017AddInTemplateSet/commit/e1a3ceb811717929b5d758cacd41e9f46917758f):
- Revit 2017 class library (C# project template).
- Revit 2017 new class (C# item template).
- The `Revit 2017 External Application` contains a special additional configuration – `Debug via Revit Add-In Manager`. It enables you to edit and debug your external command code without restarting Revit.
- Templates of all projects include a search engine to search for missing .NET assemblies. You can use the `AssemblyResolves.xml` configuration file for managing where your missing assemblies might be located.
- Video tutorial on [Debugging using the Add-In Manager](https://www.youtube.com/watch?v=QFFwG6rz0gc).
Please refer to
the [Revit2017AddInTemplateSet documentation](https://github.com/Andrey-Bushman/Revit2017AddInTemplateSet) to
see the full context and list of other video tutorials.
Many thanks to Andrey for setting up and maintaining these powerful and full-fledged templates!