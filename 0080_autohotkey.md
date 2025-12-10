---
post_number: "0080"
title: "AutoHotKey"
slug: "autohotkey"
author: "Jeremy Tammik"
tags: ['filtering', 'parameters', 'revit-api', 'views', 'windows']
source_file: "0080_autohotkey.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0080_autohotkey.html"
---

### AutoHotKey

Automate beyond the API ...
Some parts of the Revit functionality are not yet accessible through the API, even though are repetitive tasks that are trivial to perform manually and a lot of work could be saved by automating them.
One tool that many have found useful for this task and have published a number of working solutions for is
[AutoHotScript](http://www.autohotkey.com).
Here is an
[AUGI discussion](http://forums.augi.com/showthread.php?t=84683)
presenting a number of them, such as how to:

- Define F2 to open the View Range.
- Define F3 do perform Purge Unused.
- Define F5 to do a Save As to a filename that's been copied to the Windows clipboard first, and sets some additional options.
- Define F7 to set a solid's material to a new parameter called "Material".
- Define Win + 1 to Filter.
- Define Win + 2 to Finish Sketch.
- Define Win + 3 to Quit Sketch.
- Define Win + 1-4 to set various column heights.
- Automatically answer the message "This Central File has been copied".
- Remove the username from the INI file.