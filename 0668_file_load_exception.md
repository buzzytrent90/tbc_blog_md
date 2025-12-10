---
post_number: "0668"
title: "Revit Add-in File Load Exception"
slug: "file_load_exception"
author: "Jeremy Tammik"
tags: ['python', 'revit-api', 'windows']
source_file: "0668_file_load_exception.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0668_file_load_exception.html"
---

### Revit Add-in File Load Exception

Back from my vacation in Andalusia, and now moving full speed ahead towards
[Autodesk University](http://thebuildingcoder.typepad.com/blog/2011/09/revit-and-aec-api-classes-at-autodesk-university.html),
my lecture and hands-on lab on the
[Revit extensible storage API](http://thebuildingcoder.typepad.com/blog/2011/09/revit-and-aec-api-classes-at-autodesk-university.html) and all the other exciting goodies there.

Meanwhile, here is one little issue that immediately arose with the
[Revit String Search](http://thebuildingcoder.typepad.com/blog/2011/10/string-search-adn-plugin-of-the-month.html) utility
published last week:

Probably the most common problem that people keep running into with add-ins on Windows Vista, both in Revit and other environments, is the need to **unblock the zip file**.

If you are impatient, the
[solution is easy and available](http://labs.blogs.com/its_alive_in_the_lab/2011/05/unblock-net.html) with no need to read any further.

If you are interested in other aspects, here goes with another take on this:

Zach Kron had an issue on some non-XP machines trying to run Daren Thomas'
[RevitPythonShell](http://thebuildingcoder.typepad.com/blog/2011/07/python-shell-in-revit-and-vasari.html).

It was not installing properly and displaying the following message:

![FileLoadException running RevitPythonTool](img/file_load_exception_1.png)

The error message says:

Revit cannot run the external application "RevitPythonShell".
Contact the provider for assistance.
Information they provided to Revit about their identity: asdf.

System.IO.FileLoadException

Could not load file or assembly
'file:///C:\addins\Daren Thomas\RevitPythonShell\RevitPythonShell.dll'
or one of its dependencies.
Operation is not supported (Exception from HRESULT: 0X80131515)

Zach figured out the solution, which has already been documented by Gregory Mertens of
[mertens3d.com](http://www.mertens3d.com) in
the
[hatch22-2012 installation guide](http://mertens3d.com/tools/revit/2012/hatch22-2012/hatch22-2012-download-install.php),
who in turn picked it up from joseguia in this
[revitforum.org thread](http://revitforum.org/showthread.php/1793-RFO-Ribbon-Add-In-Downloads):

The issue is due to the security option.
You can right click on RevitPythonShell.dll in Win 7 and select Properties > General to obtain access to the Unblock button for setting it:

![Win 7 security option](img/file_load_exception_2.png)

Many thanks to Zach for pointing this out, and to Gregory for running into, solving, and providing the problem solution in the first place!
Hopefully this will be a helpful hint for anyone running into a similar issue installing some other Revit add-in.

We later discovered that
[Kean Walmsley](http://through-the-interface.typepad.com/through_the_interface) also
provided a comprehensive explanation of this issue to
[unblock ZIP files before installing Plugins of the Month](http://labs.blogs.com/its_alive_in_the_lab/2011/05/unblock-net.html) on
Autodesk Labs.