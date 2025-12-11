---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: qa
optimization_date: '2025-12-11T11:44:13.400564'
original_url: https://thebuildingcoder.typepad.com/blog/0123_add_in_keyboard_shortcut.html
post_number: '0123'
reading_time_minutes: 1
series: general
slug: add_in_keyboard_shortcut
source_file: 0123_add_in_keyboard_shortcut.htm
tags:
- revit-api
- walls
title: Add-In Keyboard Shortcut
word_count: 241
---

### Add-In Keyboard Shortcut

Here is a short item from a case handled by Saikat Bhattacharya:

**Question:**
How can I define keyboard shortcuts for external commands in the Revit 2010 add-ins tab?

**Answer:**
Shortcut keys for Revit are defined by adding entries to the file KeyboardShortcuts.txt.
This file is located in the Revit Program folder, together with Revit.exe and Revit.ini.
It contains a commented header section describing how to construct a new shortcut key entry.

A shortcut definition consists of a line specifying the key sequence, an action type and the action string.
The key sequence is quoted, the action type is 'ribbon' for add-ins, and the action string is the hyphen-delimited path to the ribbon item to activate.
Example:

```
"WA"  ribbon:"Home-Basic Modeling Tools-Wall"
```

Since add-ins are located in the ribbon, the action item for an add-in will always be 'ribbon'.
To specify the Add-Ins tab, you need to replace the hyphen by an underscore, because the hyphen is reserved as a separator character in the keyboard shortcut definition.
Therefore, the first item in the shortcut definition for an external command is 'Add\_Ins'.
The next items specify the panel name, which is 'External', followed by 'External Tools' for the external commands pulldown button.
So for a simple external command named XyzCommand in Revit.ini, the complete keyboard shortcut definition might be:

```
"XYZ" ribbon:"Add_Ins-External-External Tools-XyzCommand"
```