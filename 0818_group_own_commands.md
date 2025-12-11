---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:14.657484'
original_url: https://thebuildingcoder.typepad.com/blog/0818_group_own_commands.html
post_number: 0818
reading_time_minutes: 3
series: general
slug: group_own_commands
source_file: 0818_group_own_commands.htm
tags:
- csharp
- family
- parameters
- revit-api
- views
title: Group My Own Commands in the Ribbon
word_count: 574
---

﻿

### Group My Own Commands in the Ribbon

Here is an issue that has been presented numerous times in the past, and apparently needs to be highlighted again for newbies.

**Question:** I have implemented a Revit add-in which implements several commands.

Currently, I have added all my commands to the add-in manifest file, as recommended by various examples I found on the Internet.
It works and they do indeed appear in the Add-Ins ribbon tab under the 'External Tools' pull-down menu.

The problem with this is that other commands from other companies' add-ins appear in the same place and are mixed up with mine.

Is there some possibility to group my own commands somehow?

I would like to have my commands separated from the other companies' ones.

For example, is it possible to add a parent category for my commands in the 'External Tools' category, or to group them in some other way, for instance by creating a special ribbon category other than 'External Tools'?

**Answer:** Yes, there are several simple ways to group your commands separately.

You can define your own panel in the Add-Ins tab using the CreateRibbonPanel method.
This is the recommended approach.

If you really want, you can also define your own ribbon tab.
However, this is not recommended.

The former approach is demonstrated by many examples.
For instance, I implemented an absolutely minimal separate add-in tab providing one single button to demonstrate
[enabling ribbon items in zero document state](http://thebuildingcoder.typepad.com/blog/2011/02/enable-ribbon-items-in-zero-document-state.html).

The
[structural concrete setout point add-in](http://thebuildingcoder.typepad.com/blog/2012/08/structural-concrete-setout-point-add-in.html#3) is
very slightly more complex, providing two buttons :-)

The ADN training material on the Revit UI includes an entire class on the topic of ribbon customisation which covers almost every possibility provided for creating your own ribbon items.
You can either look at the
[Xtra version](http://thebuildingcoder.typepad.com/blog/2012/04/xtra-adn-revit-2013-api-training-labs.html)
([recently updated](http://thebuildingcoder.typepad.com/blog/2012/08/validate-roof-type-and-view-obj-on-android.html#3))
or the
[standard material](http://thebuildingcoder.typepad.com/blog/2012/08/validate-roof-type-and-view-obj-on-android.html#4) from the
[Revit Developer Center](http://www.autodesk.com/developrevit).

Read the documentation explaining the ribbon UI exercise, "Revit Ui Lab1 - Ribbon.doc", provided separately for C# and VB in the subdirectories DocsCS and DocsVB of the 2\_Revit\_UI\_API folder.

Sample source code is also provided separately for both C# and VB, respectively:

- 2\_Revit\_UI\_API/SourceCS/1\_Ribbon.cs- 2\_Revit\_UI\_API/SourceVB/1\_Ribbon.vb

#### Read-Only Parameter Workaround

To add a little hint for more advanced Revit API users, another frequently asked question has to do with read-only parameters.

Developers would often like to add parameters that cannot be modified to a family definition.
However, the Revit API currently does not provide any support for creating read-only parameters, unfortunately.

One way to make a parameter a little bit harder to modify anyway is to define its value via a formula instead of setting the value directly:

```
  FamilyParameter p
    = famMgr.get_Parameter( name );

  if( null == p )
  {
    p = famMgr.AddParameter( name,
      BuiltInParameterGroup.PG_GENERAL,
      ParameterType.Text, false );
  }

  if( p.StorageType
    == StorageType.String )
  {
    if( '\"' != val[0] )
    {
      val = "\"" + val + "\"";
    }
    famMgr.SetFormula( p, val );
  }
```