---
post_number: "0634"
title: "Set Copy Local to False"
slug: "copy_local_false"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'references', 'revit-api', 'selection', 'vbnet']
source_file: "0634_copy_local_false.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0634_copy_local_false.html"
---

### Set Copy Local to False

I cannot count the number of times I have pointed out that the Copy Local flag should be set to false on the Revit API assembly references.
Here are some of the numerous previous examples and explanations of this:

- A [C++ Revit add-in](http://thebuildingcoder.typepad.com/blog/2010/10/c-revit-add-in.html)- [Selection watcher using Idling event](http://thebuildingcoder.typepad.com/blog/2010/09/selection-watcher-using-idling-event.html)- The [DevTV add-in templates](http://thebuildingcoder.typepad.com/blog/2010/07/devtv-addin-templates.html)- [Pipe to conduit converter](http://thebuildingcoder.typepad.com/blog/2010/05/pipe-to-conduit-converter.html#1)- [Custom ribbon tab](http://thebuildingcoder.typepad.com/blog/2009/12/custom-ribbon-tab.html)- [Porting from C# to VB.NET](http://thebuildingcoder.typepad.com/blog/2009/07/porting-from-c-to-vbnet.html)- [Exporting a family instance to gbXML](http://thebuildingcoder.typepad.com/blog/2009/06/export-family-instance-to-gbxml.html#2)- [Application events in VB](http://thebuildingcoder.typepad.com/blog/2008/10/application-events-in-vb.html)- [Debugging a Revit add-in](http://thebuildingcoder.typepad.com/blog/2008/09/debugging-a-rev.html)- The [SDK samples solution](http://thebuildingcoder.typepad.com/blog/2008/09/the-sdk-samples.html)

This flag appears in the properties of the Revit API references.
It is also displayed in the list of the references themselves in a VB project.
Here they are set to True, which is what we want to **avoid**:

![Copy Local flag in a VB project](img/copy_local_vb.png)

In a C# project, you can right click on the Revit API references in the Visual Studio solution explorer and selecting its properties in the context menu to see and modify its current setting:

![Copy Local flag in a C# project](img/copy_local_cs.png)

You can toggle this property with a double click.

If this flag is set to True, Visual Studio will create local copies of RevitAPI.dll and RevitAPIUI.dll when compiling the plug-in and use these copies when loading them.
This confuses the debugger and Revit when running the add-in, as well as unnecessarily polluting your hard disk with copies of this multi-MB file.

To avoid having to reset this property when modifying an existing reference, for instance when migrating to a new version of the Revit API, do not delete the existing reference.
Instead, simply add the new reference to the current assembly to overwrite the old one.
The old, existing data will be updated, the new path will be stored, and the existing 'Copy Local' setting will be preserved.

By the way, the same applies to the
[AutoCAD.NET assemblies](http://through-the-interface.typepad.com/through_the_interface/2006/09/initialization_.html).

At least this post gives me a completely comprehensive description to refer to next time I have to point this out...