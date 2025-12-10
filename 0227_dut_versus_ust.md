---
post_number: "0227"
title: "DUT versus UST"
slug: "dut_versus_ust"
author: "Jeremy Tammik"
tags: ['revit-api']
source_file: "0227_dut_versus_ust.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0227_dut_versus_ust.html"
---

### DUT versus UST

After a wonderful weekend in the Swiss alps climbing the
[Diechterhorn](http://www.heinzkarrer.ch/BergtourenUrner/Diechterhorn.html), here is a
follow-up explanation by Jeremy Sawicki of Autodesk on the difference between DisplayUnitType and UnitSymbolType
in continuation of our last discussion on the
[unit suffix](http://thebuildingcoder.typepad.com/blog/2009/10/unit-suffix-and-the-projectunit-sdk-sample.html).

**Question:** What is the exact difference between DisplayUnitType and UnitSymbolType, please?

For instance, the default project unit settings on my system for a length are DUT\_Millimetres and UST\_None ... I wonder what exactly that means?

Also, can you suggest a way to obtain the display string representation shown in the user interface for either of these?

**Answer:** The simplest answer is based on the UI.
In the Format dialog box, DisplayUnitType corresponds to the Units dropdown:
![DisplayUnitType](img/DisplayUnitType.png)

And UnitSymbolType corresponds to the Unit symbol dropdown:
![UnitSymbolType](img/UnitSymbolType.png)

In most cases, the DisplayUnitType determines the actual unit, for example feet vs. meters, with a different conversion factor applying to each DisplayUnitType. UnitSymbolType affects how the units are indicated on the screen, usually with a suffix. For example, square feet can use a suffix of either SF or ft or no suffix at all. UnitSymbolType used to be called UnitSuffixType until we introduced a currency unit type, which uses prefixes in some cases, like $.

The above applies to most normal units which are displayed as a number followed by a suffix, but there are various special cases. For example, DisplayUnitType is also used to select certain kinds of units with special formatting, like Feet and fractional inches or Degrees minutes seconds.

I don't believe there is currently any API that can get the names of display unit types or unit symbol types.

Many thanks to Jeremy for this helpful clarification!