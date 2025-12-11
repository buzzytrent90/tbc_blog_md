---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: documentation
optimization_date: '2025-12-11T11:44:14.453992'
original_url: https://thebuildingcoder.typepad.com/blog/0716_level_generator_potm.html
post_number: '0716'
reading_time_minutes: 2
series: general
slug: level_generator_potm
source_file: 0716_level_generator_potm.htm
tags:
- levels
- revit-api
- rooms
- views
- walls
title: Level Generator ADN Plugin of the Month
word_count: 488
---

### Level Generator ADN Plugin of the Month

Another
[ADN Plugin of the Month](http://labs.autodesk.com/utilities/ADN_plugins/catalog) for
Revit has been published on Autodesk Labs: the February ADN Plugin of the Month is the
[Level Generator](http://labs.autodesk.com/utilities/levelgenerator) by
my Chinese colleague Joe Ye.

It generates multiple adjacent levels with one single command.

Inside Revit, go to Add-ins > External Tools > Level Generator to start the command.

The Level Generator dialogue appears:

![Level Generator](img/LevelGenerator.png)

Existing levels are listed in the table with their Index and Elevation in gray colour and are not editable.

To insert multiple new levels:

**1.** Click on the existing level above which to add new levels.

**2.** In the 'Insert multiple levels to table' area on the right-hand side of the dialog, specify the following:

- Prefix – prefix to the level number- Start Number – starting number of new levels- Suffix – suffix to the level number

These three items will define the level names.

- Level height – the height between two levels- Repeat Levels – # of levels you would like to insert

After you have specified this information, click the Insert button.
This fills in the table with the floor information.

Note: At this stage, the new levels have not yet been created in the Revit document.

The unit of the Level Height text box is the same as the current document length unit setting.

**3.** Repeat the above steps to add more levels as needed.
You can also modify the data populated in the table.
For example, you may want to modify the height of a specific level, rename levels, or delete levels.

**4.** Finally, click the OK button to generate levels according to the data in the table.

The add-in checks if all the names are unique.
If so, it creates levels, and corresponding plan and ceiling views.
![Level Generator with new levels](img/LevelGenerator2.png)

As with all ADN Plugins of the Month, full source code is provided, and the intention is for this to be both useful as is out of the box and also provide a suitable starting point for your own enhanced and personalised version.

#### Revit ADN Plugins of the Month

- March 2011 – [Room Renumbering](http://thebuildingcoder.typepad.com/blog/2011/03/room-renumbering-plugin-of-the-month.html)- April 2011 – [ASHRAE Viewer](http://thebuildingcoder.typepad.com/blog/2011/03/ashrae-viewer-plugin-of-the-month.html)- June 2011 – [File Upgrader](http://thebuildingcoder.typepad.com/blog/2011/06/file-upgrader-plugin-of-the-month.html)- August 2011 – [Wall Opening Area](http://labs.autodesk.com/utilities/wallopeningarea/)- September 2011 – [TransTips tooltip translator](http://thebuildingcoder.typepad.com/blog/2011/09/transtips-adn-plugin-of-the-month.html)- November 2011 – [String Search](http://thebuildingcoder.typepad.com/blog/2011/10/string-search-adn-plugin-of-the-month.html)

Full list of all
[ADN Plugins of the Month](http://labs.autodesk.com/utilities/ADN_plugins/catalog).