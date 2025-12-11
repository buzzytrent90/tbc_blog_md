---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.3
content_type: news
optimization_date: '2025-12-11T11:44:15.597667'
original_url: https://thebuildingcoder.typepad.com/blog/1245_net_open_source_dbg.html
post_number: '1245'
reading_time_minutes: 1
series: general
slug: net_open_source_dbg
source_file: 1245_net_open_source_dbg.htm
tags:
- revit-api
title: .NET Open Source and Visual Studio Community
word_count: 255
---

### .NET Open Source and Visual Studio Community

Steve Mycynek of the Revit Development team pointed out an important update on .NET and Visual Studio:

As you may know, one frustration faced by Revit API developers is the lack of debugging support in the free Express versions of Visual Studio.

They do not support debugging DLLs against an EXE host or attaching to a process, which is exactly what we require to debug a Revit add-in loaded into Revit.exe (except by
[resorting to tricks](http://thebuildingcoder.typepad.com/blog/2011/08/visual-studio-c-and-vb-express.html),
used, e.g., for the
[space adjacency for heat load calculation add-in](http://thebuildingcoder.typepad.com/blog/2013/07/football-and-space-adjacency-for-heat-load-calculation.html#3)).

This meant that if you wanted official support to debug a Revit add-in, you had to buy an expensive Visual Studio License – until now!

Last week, Microsoft presented a large set of free and open source announcements, including Visual Studio Community Edition, which is free and supports many more advanced features.

I just tried it out, and, as you can see, it also supports Revit add-in debugging:

![Debugging in Visual Studio Community Edition](img/vs_community.png)

Learn more at [www.visualstudio.com](http://www.visualstudio.com) and from the article by Scott Hanselman on
[.NET as open source, on Mac, on Linux and Visual Studio Community](http://www.hanselman.com/blog/AnnouncingNET2015NETasOpenSourceNETonMacandLinuxandVisualStudioCommunity.aspx).

Many thanks to Steve for exploring this and letting us know!