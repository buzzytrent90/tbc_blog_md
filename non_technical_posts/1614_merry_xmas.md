---
post_number: "1614"
title: "Merry Xmas"
slug: "merry_xmas"
author: "Jeremy Tammik"
tags: ['python', 'revit-api', 'schedules', 'selection', 'sheets', 'transactions', 'views', 'windows']
source_file: "1614_merry_xmas.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1614_merry_xmas.html"
---

### Magic Number, Magic Automation and Magic Season
Time to wind down for the year...
Here are some last things to share before signing off:
- [Magic number for schedule instance bounding box margin](#2)
- [Using PostCommand with the Windows Automation API](#3)
- [Programmer jokes](#4)
- [Season's greetings](#5)
#### Magic Number for Schedule Instance Bounding Box Margin
I discussed
the [ScheduleSheetInstance bounding box](https://forums.autodesk.com/t5/revit-api-forum/schedulesheetinstance-bounding-box/m-p/7619803)
with Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) Ignatovich in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
and the Revit development team confirmed the magic number he determined empirically for the hidden margin width:
\*\*Question:\*\* I am trying to place schedules to the view sheet in a vertical chain. The result should be something like:
![Target](img/ai_sibb_margin_target.png)
Actually, there are 2 (or more) schedules. I've already achieved what I want. I retrieve bounding box on active viewsheet of each ScheduleSheetInstance and move insertion point for each schedule.
The problem is that ScheduleSheetInstance bounding box is a bit larger than actual schedule table:
![Borders](img/ai_sibb_margin_borders.png)
I've draw borders with this Python shell code:

```
  shInst = selection[0] # a ScheduleSheetInstance should be selected to run this code

  bb = shInst.get_BoundingBox(doc.ActiveView)

  tx = Transaction(doc, "bounds")
  tx.Start()

  doc.Create.NewDetailCurve(doc.ActiveView,
    Line.CreateBound(bb.Min, XYZ(bb.Min.X, bb.Max.Y, 0)))
  doc.Create.NewDetailCurve(doc.ActiveView,
    Line.CreateBound(XYZ(bb.Min.X, bb.Max.Y, 0), bb.Max))
  doc.Create.NewDetailCurve(doc.ActiveView,
    Line.CreateBound(bb.Max, XYZ(bb.Max.X, bb.Min.Y, 0)))
  doc.Create.NewDetailCurve(doc.ActiveView,
    Line.CreateBound(XYZ(bb.Max.X, bb.Min.Y, 0), bb.Min))

  tx.Commit()
```

The distance is ~2.12 mm. I hardcoded this value in my code. It seems it works, but I don't know if it works in every possible case.
So, the question is:
Is there any way to retrieve this value via API?
\*\*Answer:\*\* The development team confirm your current approach and provide the magic constant that you were looking for:
For a schedule instance in the sheet view, we add an invisible margin around the actual schedule table.
This margin will show up when selecting the schedule instance.
The margin width is a hardcoded value in the schedule code.
It is 1/12", which is around 2.12 mm.
Right now, there is no way to retrieve this value through API as it is not exposed yet.
I think it is safe to use this hardcoded value as it has been there and remained constant for years.
This is what the schedule instance margin looks like when selecting the schedule instance.
![Schedule instances](img/ai_sibb_margin_schedule_instance.png)
You can see that the move control and the drag height control are showing at the margin bounding box.
#### Using PostCommand with the Windows Automation API
[Fausto Mendez](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/3909731) shared
a very nice example of using `PostCommand` in conjunction with
the [Windows Automation API](https://msdn.microsoft.com/en-us/library/windows/desktop/ff486375.aspx) to
simulate user input to drive the Revit user interface programmatically in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [Revit links graphic overrides](https://forums.autodesk.com/t5/revit-api-forum/revit-links-graphic-overrides/m-p/7604332):
`PostCommand` enables you to launch the Revit command.
That displays an interactive form expecting manual user input.
That input can be provided programmatically as well, like this:

```
  RevitCommandId vgCmdId = RevitCommandId.LookupCommandId("ID_VIEW_CATEGORY_VISIBILITY");

  uidoc.Application.PostCommand(vgCmdId);

  Jrn.TabCtrl _
    "Modal , Visibility/Graphic Overrides for 3D View: 3D View And Sheets Start View , 0" _
    , "AFX_IDC_TAB_CONTROL" _
    , "Select" , "Revit Links"

  Jrn.Grid _
    "ChildControl; Page , Revit Links , Dialog_Revit_RvtLinkVisibility; ID_TREEGRID_GRID" _
    , "MoveCurrentCell" , "1" , "Halftone"

  Jrn.Grid _
    "ChildControl; Page , Revit Links , Dialog_Revit_RvtLinkVisibility; ID_TREEGRID_GRID" _
    , "PartialEdit" , "1" , "Halftone" , "Yes"

  Jrn.Grid _
    "ChildControl; Page , Revit Links , Dialog_Revit_RvtLinkVisibility; ID_TREEGRID_GRID" _
    , "MoveCurrentCell" , "1" , "Halftone"

  Jrn.PushButton _
    "Modal , Visibility/Graphic Overrides for 3D View: 3D View And Sheets Start View , 0" _
    , "OK, IDOK"
```

For full details, please refer to
the [original discussion thread](https://forums.autodesk.com/t5/revit-api-forum/revit-links-graphic-overrides/m-p/7604332).
Many thanks to Fausto for demonstrating this powerful approach!
#### Programmer Jokes
To celebrate the quiet days coming, I wasted some time reading the StackOverflow list
of [best programmer jokes](https://stackoverflow.com/questions/234075/what-is-your-best-programmer-joke).
Here is a selection:
- When your hammer is C++, everything begins to look like a thumb.
- An SQL query walks into a bar and sees two tables. He walks up to them and says, 'Can I join you?'
- To understand what recursion is, you must first understand recursion.
- Q: What's the object-oriented way to become wealthy?

A: Inheritance.
- Q: How many programmers does it take to change a light bulb?

A: None; that's a hardware problem.
More Christmas related:
- Q: Why do programmers always mix up Halloween and Christmas?

A: Because Oct 31 == Dec 25!
Finally, to end it all:
- Command line Russian roulette:

```
[ $[ $RANDOM % 6 ] == 0 ] && rm -rf / || echo *Click*
```

#### Season's Greetings
A little [3D holiday greeting by Kevin Vandecar](https://kevinvandecar.github.io) using the Forge Viewer... click, zoom, pan at will:

Thank you, Kevin, for sharing this!
Merry Christmas and Happy New Year!