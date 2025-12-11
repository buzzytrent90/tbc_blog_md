---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.854845'
original_url: https://thebuildingcoder.typepad.com/blog/1363_custom_file_prop.html
post_number: '1363'
reading_time_minutes: 2
series: general
slug: custom_file_prop
source_file: 1363_custom_file_prop.md
tags:
- csharp
- elements
- family
- geometry
- python
- revit-api
- sheets
- vbnet
- views
- windows
title: Custom File Prop
word_count: 472
---

### Lunar Eclipse and Custom File Properties
Today, let's look at a Windows API question that can be of interest handling Revit documents as well, and mention my night out to watch the lunar eclipse:
- [Lunar Eclipse](#2)
- [Custom File Properties](#3)
#### Lunar Eclipse
Did you notice that we had
a [total lunar eclipse](https://en.wikipedia.org/wiki/September_2015_lunar_eclipse) early
Monday morning?
I spent Sunday night on a hill with some friends celebrating a full moon fire, then slept out beside the embers to catch it beginning around 4:50 in the morning:
![Lunar eclipse beginning](/j/photo/jeremy/2015/2015-09-28_tuellinger/026_lunar_eclipse_begin_cropped.jpg)
Here is a picture a little later, after the event, with the moon whole and intact again, setting over Basle:
![Moonset over Basle](/j/photo/jeremy/2015/2015-09-28_tuellinger/044_moonset_over_basel_cropped.jpg)
Unfortunately you cannot see the new Roche tower, recently completed, currently
the [highest building in Switzerland](http://www.guiding-architects.net/highest-high-rise-switzerland).
It is just off the picture, further left, i.e. south.
Nice experience, anyway, this 'red moon', well worth braving the cold.
#### Custom File Properties
Steve Goldsmith raised a question
on [Revit custom file properties](https://forums.autodesk.com/t5/revit-api/revit-custom-file-properties/td-p/5533067) quite a while back.
It now came up again
on [Stack Overflow](http://stackoverflow.com/questions/32735888/revit-custom-file-properties),
this time with more luck, receiving answers
from [Maxence](http://stackoverflow.com/users/200443/maxence)
and [Axel Minet](http://stackoverflow.com/users/5064688/axel-minet):
\*\*Question:\*\*
Does anyone know how to access, read or write the Revit file custom properties using VB.NET and the Revit API?
![Custom file properties](img/custom_file_properties.png)
They can be attached to both RVT and RFA files.
\*\*Answer:\*\*
These properties have nothing to do with Revit specifically.
They are standard Windows properties associated with OLE structured storage,
[COM Structured Storage](https://en.wikipedia.org/wiki/COM_Structured_Storage) and
the [Compound File Binary Format](https://en.wikipedia.org/wiki/Compound_File_Binary_Format).
They are stored
in [OLE property sets](https://msdn.microsoft.com/en-us/library/dd942421.aspx).
You can read them using the Windows API and view them using tools like
the [Structured Storage Viewer](http://www.mitec.cz/ssv.html).
The Revit API does not support them in any way whatsoever.
The Building Coder showed how to access them using other means, e.g., Python and .NET, respectively, in the discussions
on [RVT File Version](http://thebuildingcoder.typepad.com/blog/2008/10/rvt-file-version.html)
and [Open Revit OLE Storage](http://thebuildingcoder.typepad.com/blog/2010/06/open-revit-ole-storage.html).
Rod Howarth published a more extensive example
on [how to set custom file properties and attributes in C# .NET](http://blog.rodhowarth.com/2008/06/how-to-set-custom-attributes-file.html).