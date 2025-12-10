---
post_number: "0056"
title: "64 bit Revit API Issues"
slug: "64_bit_revit_api"
author: "Jeremy Tammik"
tags: ['revit-api']
source_file: "0056_64_bit_revit_api.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0056_64_bit_revit_api.html"
---

### 64 bit Revit API Issues

Successfully completed this year's conference tour, both Autodesk University and DevDays!
This is just a little note to point to some useful information that is available on issues that may occur with Revit add-ins running on the 64 bit platform.
In general, every Revit application is compiled to target any CPU and should thus be isolated from the underlying operating system by the .NET framework, enabling it to run unmodified on 32 and 64 bits.
However, some issues may still occur, and they have been analysed and discussed by
[Rod Howarth](http://rodhowarth.com)
on his
[blog](http://roddotnet.blogspot.com/2008/10/revit-64bit-revit-api.html)
and in an AUGI
[discussion thread](http://forums.augi.com/showthread.php?p=898872).