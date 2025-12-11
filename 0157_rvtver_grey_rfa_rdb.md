---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.2
content_type: qa
optimization_date: '2025-12-11T11:44:13.457650'
original_url: https://thebuildingcoder.typepad.com/blog/0157_rvtver_grey_rfa_rdb.html
post_number: '0157'
reading_time_minutes: 3
series: family
slug: rvtver_grey_rfa_rdb
source_file: 0157_rvtver_grey_rfa_rdb.htm
tags:
- doors
- family
- revit-api
- schedules
- selection
- views
- windows
title: RFA Version, Grey Commands, Family Context and RDB Link
word_count: 652
---

### RFA Version, Grey Commands, Family Context and RDB Link

Here are some recent issues that cropped up in case queries and other information:

1. [RFA file version.](#1)
2. [Greyed out commands in the ribbon.](#2)
3. [Project versus Family editor context.](#3)
4. [RDB Link Tool for Revit Family.](#4)

#### 1. RFA file version

**Question:**
When loading an RFA file, i.e. through LoadFamily, is there any way to know the Revit version that was used to create it, or at least to know that this RFA file has been converted to the current Revit version format?

**Answer:**
I am not aware of any Revit API methods for determining the Revit version used to create or last save an RFA file.
However, I did implement and discuss a stand-alone command line utility
[rvtver.py](http://thebuildingcoder.typepad.com/blog/2008/10/rvt-file-version.html)
to extract the Revit version information from a project file.
I just tested rvtver.py on an RFA file.
I selected the door family M\_Single-Flush.rfa from the folder
'C:\Documents and Settings\All Users\Application Data\Autodesk\RAC 2010\Metric Library\Doors'.
Running rvtver.py on that file reports

```
> rvtver.py M_Single-Flush.rfa
Build string found in 'M_Single-Flush.rfa'
  at offset 16470: Build: 20090224_0900
```

The
[post on rvtver.py](http://thebuildingcoder.typepad.com/blog/2008/10/rvt-file-version.html)
explains some of the background on the file structure.
If you look at the RFA file in a binary or hexadecimal editor, you may be able to glean some further information.

#### 2. Greyed out commands in the ribbon

**Question:**
In Revit 2010, what are the reasons for commands being disabled?
My commands are active when I switch to family template or family file.
In previous versions, external commands were greyed out in 3D view and family editor mode.
What could be the reasons in this case?

I've created some external commands and they appear in the menu.
The commands are disabled, i.e. greyed out, until I open a family template or family file.
Opening a new existing project does not enable the commands as it should.
What might I check as possible causes of the problem?
Note that if I open a family template or file, the commands are enabled and I can execute them.

**Answer:**
One possible reason is that the focus is not in the current view.
For example, after left clicking an item in project browser, for instance plan view, the external command item is grey.
You need to click any place in the region of the current view to make them active.
Other reasons why they might be greyed out are:

- You are in a tool other than the main selection tool.- You are in a perspective view.- You are in a schedule view.

None of these modes permit API commands to run at this time.

#### 3. Project versus Family editor context

**Question:**
My command works well when it is executed from within a project but not when executed from the family editor.
Is there any way to know if the command is being executed from the family editor or from a project context?
In any case, how can I obtain the Family object of the current family in the family editor?

**Answer:**
You can check if the current document is a Family document using the Document.IsFamilyDocument property. The owner family is accessible through Document.OwnerFamily.

Please have a look at the family creation samples in the FamilyCreation subdirectory of the Revit SDK samples folder. The most comprehensive of these in the WindowWizard sample.

Thanks to Adam Nagy for handling this case.

#### 4. RDB Link Tool for Revit Family

A brand new
[RDB Link Tool for Revit Family](http://labs.autodesk.com/utilities/revit_rdb)
is now available on
[Autodesk Labs](http://labs.autodesk.com).
In case you are interested, please check it out.