---
post_number: "0040"
title: "Visual Studio 2008"
slug: "visual_studio_2008"
author: "Jeremy Tammik"
tags: ['csharp', 'levels', 'revit-api']
source_file: "0040_visual_studio_2008.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0040_visual_studio_2008.html"
---

### Visual Studio 2008

Can I use Visual Studio 2008 to develop Revit API applications for Revit 2009?

The Revit API is .NET based, which insulates you from certain low-level details and enhances portability and interoperability compared to a more directly linked API such as C++.
Kean Walmsley just published a little note on
[obtaining and using Visual Studio 2008](http://through-the-interface.typepad.com/through_the_interface/2008/11/getting-visual.html)
in his blog on the AutoCAD.NET API.

The notes about developing .NET applications for AutoCAD apply to Revit development as well. Officially, as stated in "Getting Started Revit API 2009.doc", the only supported Revit application development environment is Microsoft Visual Studio 2005 or the Microsoft Visual Studio 2005 Express Edition. Due to the interoperability between versions you can use 2008 anyway. You may have to avoid making use of certain features, though, such as new C# 3.0 constructs or newly added libraries, since the Revit API requires the Microsoft .NET Framework version 2.0.