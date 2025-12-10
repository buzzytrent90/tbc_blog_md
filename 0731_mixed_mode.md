---
post_number: "0731"
title: "Mixed Mode C++ in Revit 2012"
slug: "mixed_mode"
author: "Jeremy Tammik"
tags: ['revit-api']
source_file: "0731_mixed_mode.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0731_mixed_mode.html"
---

### Mixed Mode C++ in Revit 2012

Here is a short note to start the week, on an issue that arose a couple of times using mixed mode C++ development in Revit 2012.
This information was originally published on the members-only
[ADN web site](http://www.autodesk.com/joinadn) as
a technical solution on the
[development environment for mixed mode C++ for Revit 2012](http://adn.autodesk.com/adn/servlet/devnote?siteID=4814862&id=16977078&linkID=4901650):

**Question:** I have a mixed mode C++ application.
When I run it on a machine where no version of Visual Studio was ever installed, I see the following error message:

Revit cannot run the external application "SimpleTest".

Contact the provider for assistance.

System.IO.FileNotFoundException:

Could not load file or assembly 'SimpleTest.dll' or one of its dependencies.
The specified module could not be found.

It seems as if some components that come with VS2010 are missing in the Revit installation.
How can I solve this, please?

**Answer:** Even though Revit 2012 uses the VS 2010 IDE and .NET 4.0, the native Revit code is compiled with the VS 2008 C++ compiler, and not with VS 2010.
When an add-in makes use of unmanaged C++ code built with VS 2010 components, those components are not provided by the Revit installation.

There are two ways to solve this: you can either (1) set up your native C++ projects to use the VS 2008 compiler, or (2) install the VS 2010 redistributables yourself together with your add-in.