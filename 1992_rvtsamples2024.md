---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:17.219910'
original_url: https://thebuildingcoder.typepad.com/blog/1992_rvtsamples2024.html
post_number: '1992'
reading_time_minutes: 7
series: general
slug: rvtsamples2024
source_file: 1992_rvtsamples2024.md
tags:
- elements
- geometry
- parameters
- revit-api
- sheets
- transactions
- vbnet
- views
- walls
- windows
title: Rvtsamples2024
word_count: 1458
---

### Configuring RvtSamples 2024 and Big Numbers
I left the Nice APS accelerator [APS cloud accelerator](https://aps.autodesk.com/accelerator-program) and
am returning to Switzerland, using the long train ride time to continue my Revit 2024 migration process,
now addressing the RvtSamples external application ad-in:
- [Configuring RvtSamples 2024](#3)
- [DatumsModification](#4)
- [ContextualAnalyticalModel](#5)
- [Infrastructure Alignments](#6)
- [Toposolid](#7)
- [Conclusion](#8)
- [Consuming huge numbers of element ids](#9)
#### Configuring RvtSamples 2024
Now that I completed installing Revit 2024,
[successfully compiled the Revit 2024 SDK samples](https://thebuildingcoder.typepad.com/blog/2023/04/nice-accelerator-and-compiling-the-revit-2024-sdk.html)
and updated the [RevitSdkSamples repository](https://github.com/jeremytammik/RevitSdkSamples),
the time is ripe to configure the RvtSamples external application to load all 246 Revit 2024 SDK sample external commands.
Yes, 246 of them.
Pretty hard to manage one by one.
Mainly, this consists of editing RvtSamples.txt, the input text file specifying the name and location of the commands and the .NET assembly DLLs implementing them.
Here is an overview of (most of) the history of RvtSamples, including its initial implementation and similar migration efforts in the past:
- [Loading SDK Samples](https://thebuildingcoder.typepad.com/blog/2008/09/loading-sdk-sam.html)
- [Adding `#include` functionality](https://thebuildingcoder.typepad.com/blog/2008/11/loading-the-building-coder-samples.html)
- [RvtSamples Conversion from 2009 to 2010](http://thebuildingcoder.typepad.com/blog/2009/05/porting-the-building-coder-samples.html)
- [Debugging with Visual Studio 2010 and RvtSamples](http://thebuildingcoder.typepad.com/blog/2010/04/debugging-with-visual-studio-2010-and-rvtsamples.html)
- [Migrating the Building Coder Samples to Revit 2012](http://thebuildingcoder.typepad.com/blog/2011/04/migrating-the-building-coder-samples-to-revit-2012.html)
- [Compiling the Revit 2014 SDK](http://thebuildingcoder.typepad.com/blog/2013/04/compiling-the-revit-2014-sdk.html)
- [Compiling the Revit 2015 SDK and Migrating Bc Samples](http://thebuildingcoder.typepad.com/blog/2014/04/compiling-the-revit-2015-sdk-and-migrating-bc-samples.html)
- [Migrating The Building Coder Samples to Revit 2016](http://thebuildingcoder.typepad.com/blog/2015/05/migrating-the-building-coder-samples-to-revit-2016.html)
- [RvtSamples for Revit 2017](http://thebuildingcoder.typepad.com/blog/2016/04/rvtsamples-for-revit-2017.html)
- [The Building Coder Samples 2017](http://thebuildingcoder.typepad.com/blog/2016/05/the-building-coder-samples-2017.html)
- [RvtSamples for Revit 2018](http://thebuildingcoder.typepad.com/blog/2017/05/sdk-update-rvtsamples-and-modifying-grid-end-point.html)
- [RvtSamples 2019](http://thebuildingcoder.typepad.com/blog/2018/04/rvtsamples-2019.html)
- [RvtSamples 2019 Update](http://thebuildingcoder.typepad.com/blog/2018/05/installing-the-revit-2019-sdk-april-update.html)
- [RvtSamples 2020](https://thebuildingcoder.typepad.com/blog/2019/04/the-revit-2020-fcs-api-and-sdk.html)
- [Close Doc and Zero Doc RvtSamples](https://thebuildingcoder.typepad.com/blog/2019/04/close-doc-and-zero-doc-rvtsamples.html)
- [RvtSamples 2020.1](https://thebuildingcoder.typepad.com/blog/2019/09/whats-new-in-the-revit-20201-api.html#4)
- [Setting up RvtSamples for Revit 2021](https://thebuildingcoder.typepad.com/blog/2020/05/setting-up-rvtsamples-for-revit-2021.html)
- [Revit 2022 SDK and The Building Coder Samples](https://thebuildingcoder.typepad.com/blog/2021/04/revit-2022-sdk-and-the-building-coder-samples.html)
- [Compiling the Revit 2023 SDK Samples](https://thebuildingcoder.typepad.com/blog/2022/04/compiling-the-revit-2023-sdk-samples.html)
I already described how to handle some of the errors encountered in previous migration cycles listed above.
Here is an overview of the problematic add-ins this time around:
#### DatumsModification
Correct list of external commands for the DatumsModification add-in:
- DatumAlignment
- DatumPropagation
- DatumStyleModification
#### ContextualAnalyticalModel
The SDK source code implements the following 21 ContextualAnalyticalModel external commands:
- Use `grep "class.\*IExternalCom" \*cs`
- AddAssociation
- AddCustomAssociation
- AnalyticalNodeConnStatus
- CreateAnalyticalPanel
- CreateAnalyticalCurvedPanel
- CreateAnalyticalMember
- CreateAreaLoadWithRefPoint
- CreateCustomAreaLoad
- CreateCustomLineLoad
- CreateCustomPointLoad
- FlipAnalyticalMember
- MemberForcesAnalyticalMember
- ModifyPanelContour
- MoveAnalyticalMemberUsingElementTransformUtils
- MoveAnalyticalMemberUsingSetCurve
- MoveAnalyticalNodeUsingElementTransformUtils
- MoveAnalyticalPanelUsingElementTransformUtils
- MoveAnalyticalPanelUsingSketchEditScope
- ReleaseConditionsAnalyticalMember
- RemoveAssociation
- SetOuterContourForPanels
These are the ContextualAnalyticalModel external commands listed in RvtSamples.txt:
- Use `grep "^ContextualAnalyticalModel" RvtSamples.txt | sort`
- AddRelation
- AnalyticalNodeConnStatus
- BreakRelation
- CreateAnalyticalCurvedPanel
- CreateAnalyticalMember
- CreateAnalyticalPanel
- FlipAnalyticalMember
- MemberForcesAnalyticalMember
- ModifyPanelContour
- MoveAnalyticalMemberUsingElementTransformUtils
- MoveAnalyticalMemberUsingSetCurve
- MoveAnalyticalNodeUsingElementTransformUtils
- MoveAnalyticalPanelUsingElementTransformUtils
- MoveAnalyticalPanelUsingSketchEditScope
- ReleaseConditionsAnalyticalMember
- SetOuterContourForPanels
- UpdateRelation
#### Infrastructure Alignments
- Infrastructure Alignment Station Label
- Infrastructure Alignment Properties
#### Toposolid
The Toposolid sample only has one entry in RvtSamples.txt specifying an external command named:
- Revit.SDK.Samples.Toposolid.CS.Command
This command does not exist.
Instead, the sample implements the following external commands:
- ToposolidCreation
- ToposolidFromDWG
- ContourSettingCreation
- ContourSettingModification
- ToposolidFromSurface
- SSEPointVisibility
- SplitToposolid
- SimplifyToposolid
#### Conclusion
This time around, I submitted a ticket with the development team in the hope of avoiding having to repeat this entire process for the next SDK update:
- REVIT-206304 – Update RvtSamples.txt for Revit 2024 SDK
Some of the menus generated by RvtSamples had too many entries to display them all on my screen, so I modified the sorting and added two new groups for `Analytical Model` and `Toposolid`.
![RvtSamples 2024](img/rvtsamples2024.png "RvtSamples 2024")
My current running version of RvtSamples is captured
in [RevitSdkSamples release 2024.0.0.3](https://github.com/jeremytammik/RevitSdkSamples/releases/tag/2024.0.0.3).
#### Consuming Huge Numbers of Element Ids
The [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [draw a line visible on screen](https://forums.autodesk.com/t5/revit-api-forum/draw-line-visible-on-screen/m-p/11922998) spawned
several useful ideas on how to generate and display transient graphics for rubber banding functionality similar to AutoCAD jigs.
It also raised the question of generating (and consuming) huge amounts of element ids, since each transient element in a loop consumes a new element id, even if the transaction is never commited.
Luckily, Revit 2024 [upgraded `ElementId` storage to 64-bit](https://thebuildingcoder.typepad.com/blog/2023/04/whats-new-in-the-revit-2024-api.html#4.1.2).
Anyway, that question led to the following amusing discussion:
- An interesting question was raised in the discussion on drawing transient geometry: if I create a new element in a transaction that is rolled back (not committed) and wrap that in a loop, a new element id is consumed in each iteration. That can consume all of the element ids space if I run it for long enough. How should this be handled, please?
- This is a non-issue in Revit 2024.+. Element id is now 64 bit.
- That does not make it a non-issue. It just means it takes a million or a billion time more time to consume all the possible tokens. Is that a real solution?
- `LLONG_MAX` is 9,223,372,036,854,775,807. We won't need to worry about it.
Should the next element id field in document be rolled back on transaction rollback is an independent issue though.
Another independent issue is should the editor rubber banding reuse the same element ids on every iteration?
- Well, if I rubber band a line in a loop and create a new element id for every iteration, and assuming an extremely slow loop running 100 iterations per second, I will end up using 100\*60\*60\*24\*365 = 3.153.600.000 element ids in one year of rubber banding... that is not a completely insignificant number...
- Jeremy, is that three trillion?
- It's three thousand million. I don't know the exact definition of a trillion, but it's a big number :-)
- Ok. the new Element id max is 9 quintillion. Dividing 9 quintillion by your number gives me 3 million, which would be the number of years required for your example to run through the ElementId space. That's why we've asserted that the 64-bit ids are unlikely to run out any time soon.
- To do another example, if we had an operation running 24/7 which generated 1 million new ids per second:
- 1,000,000 \* 60 \* 60 \* 24 \* 365 = 3.1536e+13
- 9,223,372,036,854,775,807 / 3.1536e+13 = 292471 (plus a fraction)
- which is just a wild number of years.
- It is worth noting that we tried doing this and estimated that it would take days to even get to the end of the 32-bit space by creating the simple transient elements. It would therefore take on the order 2^31 days to run out of 64-bit ids.
So, to confirm what is said above, it would take an extremely long time to run out of the newly extended elementid space
- To pick up on the 3 million years mentioned above: Revit will survive 3 million years, but the rubber bands will surely lose some elasticity.
- Can I just say all of you are awesome and this conversation makes me so happy I work here :-)
For some related facts, consult the illuminating ten-minute video by Numberphile explaining [what is a billion?](https://youtu.be/C-52AI_ojyQ).