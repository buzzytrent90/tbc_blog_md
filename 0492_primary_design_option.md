---
post_number: "0492"
title: "Primary Design Option"
slug: "primary_design_option"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api']
source_file: "0492_primary_design_option.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0492_primary_design_option.html"
---

### Primary Design Option

Here is a neat little
[idea by Benson](http://thebuildingcoder.typepad.com/blog/2009/11/visible-elements.html?cid=6a00e553e168978833013488fe0388970c#comment-6a00e553e168978833013488fe0388970c) on
how to retrieve the primary design option in a project that has been rattling around in my to-do list for a while now and seems suitable for a Tel Aviv Saturday morning post:

The method DesignOption.GetActiveDesignOptionId exists in the Revit 2011 API, and this method can return the 'active' design option id.
However, it is not the primary design option id.

The DesignOption class property IsPrimary indicates whether a design option is primary.

So, we can iterate though all design options and use that property to determine the primary one.

The Revit 2010 API does not provides the method 'GetActiveDesignOptionId' and property 'IsPrimary', so it seems impossible to get the active design option and primary option in that version.

Thanks to Benson for this hint!