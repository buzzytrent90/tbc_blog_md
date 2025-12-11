---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.100023'
original_url: https://thebuildingcoder.typepad.com/blog/0523_ui_auto_design_option.html
post_number: '0523'
reading_time_minutes: 3
series: general
slug: ui_auto_design_option
source_file: 0523_ui_auto_design_option.htm
tags:
- csharp
- elements
- revit-api
- views
title: Automate DesignOption and 64 Bit Add-In Templates
word_count: 631
---

### Automate DesignOption and 64 Bit Add-In Templates

Let's start off the week with another idea from Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de).
He recently presented his results of exploring the Revit ribbon internals using UISpy,
[driving Revit using UI Automation](http://thebuildingcoder.typepad.com/blog/2011/01/ribbon-spying-and-ui-automation.html),
[subscribing to UI Automation events](http://thebuildingcoder.typepad.com/blog/2011/01/subscribing-to-ui-automation-events.html), and
[switching between different projects and views](http://thebuildingcoder.typepad.com/blog/2011/01/further-ideas-for-using-ui-automation.html).

Here is another of his ideas of making use of UI Automation:

This screen snapshot shows the design options dropdown list:

![Design options dropdown list](img/design_option.png)

When showing elements programmatically using the app.ActiveUIDocument.ShowElements( elementSet ) method, it might occur that some elements cannot be shown because their design option is currently invisible.

The current design option visibility is defined per document and affects all views.

UI Automation could be used to set the current design option to the selected element's one, so it can be made visible.

So, if you want to show a given element, you can compare its design option with the current one before showing it.

Of course this is only possible if all elements in the element set passed in the ShowElements have the same design option.

If you just want to show a single element, this will never be a problem.

Yet again, many thanks to Rudolf for his many ideas and these valuable pointers!

#### 64-bit Visual Studio DevTV Revit Add-in Template

Stephen LeCompte posted a
[comment](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html?cid=6a00e553e1689788330148c784cbbc970c#comment-6a00e553e1689788330148c784cbbc970c) on the
[DevTV add-in templates](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html) which
integrate with the Visual Studio new project wizard to automatically set up a new Visual Studio project for a Revit add-in complete with skeleton external application, external command, and add-in manifest code.
By the way, there is also a streamlined and
[updated C# version](http://thebuildingcoder.typepad.com/blog/2010/10/revit-2011-devtv.html#2) for
my personal use with comments removed and other tweaks.

Stephen worked on making use of the templates on a 64 bit system and ran
[into](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html?cid=6a00e553e1689788330148c77b8fd9970c#comment-6a00e553e1689788330148c77b8fd9970c)
[some](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html?cid=6a00e553e1689788330147e171f013970b#comment-6a00e553e1689788330147e171f013970b)
[problems](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html?cid=6a00e553e1689788330147e172522d970b#comment-6a00e553e1689788330147e172522d970b)
which were finally
[resolved](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html?cid=6a00e553e1689788330148c784cbbc970c#comment-6a00e553e1689788330148c784cbbc970c):

OK, I did eventually find a solution to my problem where I do not have 32-bit Revit version installed but only 64-bit Revit.

After downloading the winzip file above...

1. extract the contents- within the .csproj file...
     1. do a find/replace on $(ProgramFiles) and replace with C:\Program Files- find any instances where processorArchitecture=x86 and replace with processorArchitecture=x64- delete the old winzip file- create a new .zip that combines all the files completed.

The initial problem was trivial, forgetting to delete the original winzip even though the appropriate changes were made.

Here are Stephen's 64 bit templates:

- [Template64BitOnlyRevitArchAddInCs.zip](zip/Template64BitOnlyRevitArchAddInCs.zip)- [Template64BitOnlyRevitArchAddInVb.zip](zip/Template64BitOnlyRevitArchAddInVb.zip)

They have been set up so that if you download the first templates and they do not work, you can still simply download this second version without deleting the old ones and see the new ones clearly as Revit Architecture 2011 64-bit only Addin.

Many thanks to Stephen for solving the issue and sharing his files with us!