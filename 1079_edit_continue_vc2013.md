---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.7
content_type: documentation
optimization_date: '2025-12-11T11:44:15.262613'
original_url: https://thebuildingcoder.typepad.com/blog/1079_edit_continue_vc2013.html
post_number: '1079'
reading_time_minutes: 1
series: general
slug: edit_continue_vc2013
source_file: 1079_edit_continue_vc2013.htm
tags:
- revit-api
- schedules
- views
title: Visual Studio 2013 May Partially Support Edit and Continue
word_count: 279
---

### Visual Studio 2013 May Partially Support Edit and Continue

The West European DevDays conferences are completed.
I am on my way back from Gothenburg to Switzerland.
En route, I would like to share this strange and useful discovery by Peter Muigg of
[CAD Anwendungen Muigg](http://www.muigg.com), who says:

In spite of what was said in the article on
[debugging Revit 2014 API with Visual Studio 2013](http://thebuildingcoder.typepad.com/blog/2013/11/debugging-revit-2014-api-with-visual-studio-2013.html) and
the
[Revit API discussion thread](http://forums.autodesk.com/t5/Revit-API/Debugging-Revit-2014-API-with-Visual-Studio-2013/td-p/4574097) on
the same topic, Visual Studio 2013 does in fact partially support edit and continue.

Just as stated there, the add-in ribbon items are disabled when you launch Revit in the Visual Studio 2013 debugger.

However, that is only true in the graphical views.

If you switch to a schedule view, they become enabled, the add-in can be launched, debugged, and the edit and continue feature works.

If you absolutely require running the add-in in a graphical view, you have to start Revit from Visual Studio without debugging and then attach to process afterwards, in which case the edit and continue functionality is lost again.

I have not personally been able to test any of this, not having Visual Studio 2013 installed, but would love to hear from others if they can confirm these findings.

Thank you, Peter, for this exciting discovery, and thank you, reader, for testing and confirming.

Now to get back home and start my Christmas
[Guetzli](http://als.wikipedia.org/wiki/Keks) baking.