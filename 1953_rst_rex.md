---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.3
content_type: qa
optimization_date: '2025-12-11T11:44:17.120392'
original_url: https://thebuildingcoder.typepad.com/blog/1953_rst_rex.html
post_number: '1953'
reading_time_minutes: 6
series: general
slug: rst_rex
source_file: 1953_rst_rex.md
tags:
- elements
- parameters
- references
- revit-api
- sheets
- views
title: Rst Rex
word_count: 1238
---

### Structural Questions and the Future of REX
Today, we ponder some structural rebar questions and the future of the REX structural Revit extensions included with the Revit SDK:
- [Future of REX](#2)
- [Rebar API questions](#3)
- [GetCustomDistributionPath](#4)
- [Number of segments](#5)
- [IsRebarInSection](#6)
#### Future of REX
Tomasz Wojdyla joined the Robot Structural Analysis team as senior product owner and is wondering about the future of the Revit Extension Framework, aka. REX.
We currently support it on a yearly basis even though (we believe) there is no internal use of this component anymore.
External use is a question mark – it would be great if we could collect some data about its (external) users.
Can you please let us know if you are interested in this framework?
You can simply provide your details in a comment below, or send
a [private message](https://thebuildingcoder.typepad.com/blog/about-the-author.html#1) if you prefer that.
All info and advice is greatly appreciated!
Thank you!
#### Rebar API Questions
Miguel [MiguelGT17](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/5130624) Gutierrez raised a number of rebar questions in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) that were very kindly answered
by Stefan Dobre, ‪senior principal engineer of the Revit Structural development team:
He responded: After a quick look on these questions, I don't see any problems.
It is just a misunderstanding of the API and how the rebar works.
I answered in the forum and submitted a change request \*REVIT-191469\* to update the description and (maybe) the name of the `Rebar.IsRebarInSection` function.
Many thanks to Stefan for the quick solutions!
#### GetCustomDistributionPath
First, how to
call [GetCustomDistributionPath from RebarFreeFormAccessor](https://forums.autodesk.com/t5/revit-api-forum/getcustomdistributionpath-from-rebarfreeformaccessor/td-p/11148790)?
\*\*Question:\*\* Is there a way to group up rebars without using the `RebarContainer` command and loading the distribution path data?
![Distribution path](img/mg_distributionpath_1.png "Distribution path")
Furthermore, there is something strange going on when creating a rebar container.
Its distribution path is not correlated to the actual true distribution path:
![Distribution path](img/mg_distributionpath_2.png "Distribution path")
\*\*Answer to Question 1:\*\* There are two types of Free Form rebar:
The one created from curves, and which doesn’t have any constraints to the host.
The input curves for each bar in set can be in any position and it is not possible to set a distribution path for it.
An example of such rebar is the sketched free form.
To create such Rebar you should call
```csharp
Rebar CreateFreeForm( Document doc,
RebarBarType barType, Element host,
IList curves,
out RebarFreeFormValidationResult error);
```
The one created through a server (callback), and which has constraints to the host.
Any time when the constraints are modified, the server is called to recompute the Rebar curves.
During this calculation, a distribution path can be set too.
This distribution path is a list of curves.
Aligned and Surface distributions are examples of such free form.
To create such Rebar, you should use
```csharp
Rebar CreateFreeForm( Document doc,
Guid serverGUID, RebarBarType barType,
Element host);
```
Look also at the documentation for `IRebarUpdateServer` class.
The Revit SDK includes a sample that demonstrates how such a free form rebar can be created.
\*\*Answer to Question 2:\*\* `RebarContainerItem` is created from a Rebar element (which is a set).
It has its own number of bars and of course it will have its own distribution path which is the source rebar’s distribution path.
`RebarContainer` is just storing a list of RebarContainerItems without maintaining any relations between them.
\*\*Response:\*\* Appreciate those comments;
indeed, they are very deep insights about question 1.
Concerning question 2 reply, I'm worried about it.
I'll find the best approach to reach my goal with the information you have provided so far.
Stay blessed.
#### Number of Segments
Next, Miguel raises a question on
the [number of segments](https://forums.autodesk.com/t5/revit-api-forum/number-of-segments/td-p/11148840):
\*\*Question:\*\* This set of rebars has been sketched as a free form.
![Number of segments](img/mg_nr_segments_1.png "Number of segments")
RevitLookup shows a single segment for this bar:
![Number of segments](img/mg_nr_segments_2.png "Number of segments")
This is not true:
![Number of segments](img/mg_nr_segments_3.png "Number of segments")
Moreover, the `IsRebarInSection(view)` command always return false, regardless of the view.
So, I am not going to have the appropriate data when I sketch rebars as freeform?
\*\*Answer:\*\* As I can see in your images, you have a free form rebar that has the Workshop Instructions parameter set to Keep Straight. This means that no matter what curves the free form has, it will always be matched with a straight shape (M_00).
If you want the bar to be matched with other shapes, you should set the workshop parameter to Bend. One you set this option, each bar in the set will be matched with the existing rebar shapes from the project. If it doesn't match with any existing shapes, it will try to create new Rebar Shapes. If it can’t create new Rebar Shapes and error message will be posted and will continue to consider the bar as a straight one.
For more details on how the shape matching is working you can have a look on this: https://knowledge.autodesk.com/support/revit/learn-explore/caas/CloudHelp/cloudhelp/2019/ENU/Revit-M...
Rebar.IsRebarInSection(View view) returns true only if the view is a section or elevation and the view plane is cutting at least one of the rebar curves, false otherwise. This API function is the correspondent of this UI option:
![IsRebarInSection](img/mg_nr_segments_4.png "IsRebarInSection")
\*\*Response:\*\* Thanks for your prompt reply; my bad, I was not aware of those parameters.
I will double check them and perform another test this weekend.
Cheers!
#### IsRebarInSection
Finally, on [`IsRebarInSection`](https://forums.autodesk.com/t5/revit-api-forum/isrebarinsection/td-p/11148854):
\*\*Question:\*\* I have placed a set of rebars in a viewPlan that only has 1 segment:
![IsRebarInSection](img/mg_isrebarinsection.png "IsRebarInSection")
I was expecting the `IsRebarInSection` method to return a `true` Boolean, as the rebars are shown as a cross section.
If that is not the case, what does this method stand for, and which API method should I be looking up instead?
Test code:
```csharp
foreach (Element element in rebars)
{
if (element is RebarContainer)
{
}
else if (element is Rebar)
{
Rebar el = element as Rebar;
if (el.IsRebarFreeForm() == true)
{
}
else if (el.IsRebarShapeDriven() == true)
{
RebarShapeDrivenAccessor acc = (element as Rebar).GetShapeDrivenAccessor();
XYZ dir = acc.GetDistributionPath().Direction;
double angle = dir.AngleTo(view.ViewDirection);
angle = angle \* (180 / Math.PI); //90,0
stb.AppendLine(angle.ToString());
stb.AppendLine(el.IsRebarInSection(view).ToString());
}
}
TaskDialog.Show("dd", stb.ToString());
}
```
\*\*Answer:\*\* `IsRebarInSection` returns true only if the view is a section or elevation and the view plane is cutting at least one of the rebar curves, false otherwise.
This API function is the correspondent of this UI option:
![IsRebarInSection](img/mg_nr_segments_4.png "IsRebarInSection")
In your case, to see that the straight bar is shown as a point you can verify this on your own.
You can get the centerline curves like this:
```csharp
rebar.GetTransformedCenterlineCurves(
false, true, true,
MultiplanarOption.IncludeOnlyPlanarCurves,
0);
```
You will get only one line. If the line’s direction is parallel with view’s direction it means that the bar is shown as a cross section, false otherwise.