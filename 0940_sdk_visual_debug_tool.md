---
post_number: "0940"
title: "Revit 2014 SDK and Visual Debugging Tools"
slug: "sdk_visual_debug_tool"
author: "Jeremy Tammik"
tags: ['elements', 'geometry', 'revit-api', 'rooms', 'views', 'walls']
source_file: "0940_sdk_visual_debug_tool.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0940_sdk_visual_debug_tool.html"
---

### Revit 2014 SDK and Visual Debugging Tools

Two hot topics for today, especially the first:

- [Updated Revit 2014 SDK](#2)
- [Visual debugging tools](#3)

#### Updated Revit 2014 SDK

An updated Revit 2014 SDK is live on the ADN Open
[Revit developer page](http://www.autodesk.com/developrevit) now:

[www.autodesk.com/developrevit](http://www.autodesk.com/developrevit)

This version includes RevitLookup and the AddInManager!

#### Visual Debugging Tools

Rudolf Honke of
[Mensch und Maschine acadGraph GmbH](http://www.acadgraph.de) discovered
a glitch in the definition of the room solids in one of the standard sample RVT project files.

**Rudi says:** When testing VRML exporter with Revit 2014, I noticed that the brand new rac\_sample\_project.rvt has some inconsistencies.

I export the ClosedShells of the rooms into VRML format to display then in my browser.
On the left side you see overlapping room volumes from 'Hall' and 'Entry Hall':

![Overlapping room volumes](img/visual_debugging_1.png)

'Kitchen and dining' has an upper edge that needs to be cut off:

![Upper edge sticks out](img/visual_debugging_2.png)

'Hall' has two ears protruding into the sky:

![Hall with ears](img/visual_debugging_3.png)

I noticed this when debugging the funny results my application was producing.

Since it is not the code, I concluded it must be the input that was corrupted…

Without a visual tool like this, it is quite hard to evaluate what's wrong.

**Jeremy adds:** You can also observe this using the Export > gbXML command:

![Overlapping room volumes](img/visual_debugging_4.png)

'Kitchen and dining' has an upper edge that needs to be cut off:

![Upper edge sticks out](img/visual_debugging_5.png)

'Hall' has two ears protruding into the sky:

![Hall with ears](img/visual_debugging_6.png)

Furthermore, the Revit 2014 program folder contains a utility named **gbXML2dwfx.exe**.

If you drag and drop a gbXML file onto this it will create a DWFx file that can be viewed in
[Autodesk Design Review](http://usa.autodesk.com/design-review),
the free 2D and 3D DWF viewer, which provides more functionality than many VRML viewers.

**Response:** Interesting.
GbXML2Dwfx.exe appears to be new in Revit 2014.

Regarding the geometry, it is important to note that you cannot rely on geometrical integrity of your Revit Elements.

I faced this problem previously, not only with rooms but also with walls.

Every cut-off and every Boolean operation makes the geometry more complex, and rounding errors may occur.

This totally confirms the GiGo principle of
[Garbage in, garbage out](http://en.wikipedia.org/wiki/Garbage_in,_garbage_out),
slightly less helpful than
[FiFo](http://en.wikipedia.org/wiki/FIFO)...