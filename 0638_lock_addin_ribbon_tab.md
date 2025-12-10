---
post_number: "0638"
title: "Locking the Add-Ins Ribbon Tab"
slug: "lock_addin_ribbon_tab"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'selection']
source_file: "0638_lock_addin_ribbon_tab.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0638_lock_addin_ribbon_tab.html"
---

### Locking the Add-Ins Ribbon Tab

Here is a minute user interface question of interest to developers that comes up from time to time:

**Question:** When I select an element in the Revit model, the ribbon tab is always switched.
Is there any way to keep 'Add-Ins' tab focused at all times?

**Answer:** Here are two little tricks that might help:

- Drag the panel you wish to have available off the ribbon add-ins tab somewhere else on the screen.
  It will then remain available at all times, regardless of how the other ribbon tabs are switched back and forth.- As I already mentioned discussing the
    [ribbon tab context toggle](http://thebuildingcoder.typepad.com/blog/2010/07/context-tab-toggle.html),
    you can also select Big R > Options > User Interface and uncheck 'Display the contextual tab on Selection':

![Ribbon tab context selection option](img/context_tab_toggle.png)