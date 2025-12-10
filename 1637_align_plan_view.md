---
post_number: "1637"
title: "Align Plan View"
slug: "align_plan_view"
author: "Jeremy Tammik"
tags: ['elements', 'revit-api', 'sheets', 'views']
source_file: "1637_align_plan_view.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1637_align_plan_view.html"
---

### Boston Forge Accelerator and Aligning Plan Views
Here is a suggestion made by Arkady Gilman, Sr. Software Architect in the Revit development team, to align plan views.
Before we get to that, I'd like to point out the imminent Forge accelerator in Boston:
- [Boston Forge accelerator](#2)
- [Question on aligning plan views](#3)
- [Answer, view origin and outline](#3)
- [Suggestion for aligning views](#4)
- [Addendum – response](#5)
#### Boston Forge Accelerator
The Forge team is accepting proposals for the upcoming Accelerator taking place in the Autodesk Boston office next month, from April 30 - May 4.
It is a great opportunity for your software development team to learn and work intensively on a Forge-based project with one-on-one support and advice from Autodesk Forge API experts. If you have an idea for a new web or mobile application based on Autodesk Forge APIs, or you need help getting an existing application working, then come along to the event. There is no cost to attend the accelerator, other than your own travel and living expenses. However, we do ask that you submit a proposal as part of your application so that we can verify that the intended use of the Autodesk Forge APIs in your project is feasible.
Here is [how to apply](http://autodeskcloudaccelerator.com/apply).
For more information, please visit the [Forge Accelerator website](http://autodeskcloudaccelerator.com/forge-accelerator).
#### Question on Aligning Plan Views
I encountered an unexplained offset in the paperspace origin of a small number of Revit plan views as reported by the API method `ViewPlan.Outline`.
I use it to align multiple views across sheets by setting the view position to the same position relative to the sheet title block. For plan views, the alignment uses model coordinates, so the same part of the model appears in the same position relative to sheet the title block.
We align plan views across sheets by calculating the location of viewport plan view model in paperspace coordinates relative to the sheet title block.
That works for most of the plans on sheets. However, for some, it does not align the views on the sheet; they were offset by a mysterious amount.
Further investigation showed that these non-aligning views had a non-zero value for `ViewPlan.Origin`, and that value exactly accounted for the mysterious offset.
I modified my approach to accommodate the view origin offset as follows:
```csharp
XYZ GetViewportLocation( Viewport viewport )
{
. . .
// Get view origin in scaled (paperspace) coordinates
// - used to correct for mysterious origin offset
double scale = Convert.ToDouble( plan.Scale );
XYZ viewPaperspaceOrigin = new XYZ(
plan.Origin.X / scale,
plan.Origin.Y / scale, 0.0 );
// Calculate origin of viewport view in paperspace
// coordinates relative to sheet titleblock
location = viewportPaperspaceLoc - viewPaperspaceLoc
- viewPaperspaceOrigin + paddingVec;
return location;
}
```
Although `ViewPlan.Origin` seems to correct for the mysterious origin offset, I am not sure I am using a sanctioned API method.
The API documentation states that 'The origin of a plan view is not meaningful'.
By the way, I used an earlier article
discussing [sheet to model coordinate conversion](http://thebuildingcoder.typepad.com/blog/2015/10/sheet-to-model-coordinate-conversion.html) to
figure out the sheet to model transform and the .01 foot viewport buffer.
#### Answer, View Origin and Outline
It seems that you figured out correct relation between `View.Outline` and `View.Origin`.
However, the description of the `View.Origin` property in the API documentation does not sound right.
Instead of \*Returns the origin of the screen\*, it should say something like \*Model coordinates of the point in the view projection plane that serves as the origin of the paper coordinates of the view. For plan views Z-coordinate of the origin is not well defined and should not be relied upon\*.
Here are some details:
`View.Origin` is a handwritten API method, along with `View.Outline` and other methods. `View.Outline` returns 2D outline of the objects in the view in 'projection coordinates' of this view, described in the API documentation as the 'paper space' of the view. These coordinates include view scale and the outline accounts for view regions.
`View.Origin` gets the origin from the `BoundedSpace` of the corresponding `Viewer`. I suspect that the comment about origin being undefined for plans stems from the absence of special `Viewer` subclass for plan views. However, the plan view has a viewer, which is an instance of the `Viewer` base class. `BoundedSpace` is part of this base class viewer, so whatever transform applies to the view element as it is moved, rotated, mirrored also applies to that origin. Note that the origin is in model space, not in 'paper space'; this explains why dividing it by `viewScale` works.
Although the use of View.Origin in your code sample looks technically correct, I am not sure that `View.Origin` is actually needed for the task you are trying to solve. In my mind, it should not matter where the origin of paper space is on each view. I would rather not have the `View.Origin` property exposed in the API at all. I'm not sure why somebody decided to expose it, because internally we are trying not to use any such thing in Revit code.
#### Suggestion for Aligning Views
If I would try to automatically place some view, PlanB, on a sheet, that already contains PlanA, in such a way that PlanB aligns with PlanA, I would first place PlanB at an arbitrary place. Then I would take some imaginary vertical line in the model, compute what point on the sheet it ends up as visible in PlanA and in PlanB, and then compute the vector to shift PlanB's viewport on the sheet to get these two points aligned (vertically or horizontally). This is oversimplification, of course, but there is no place where view origin is required or matters.
Many thanks to Arkady for this good idea!
#### Addendum – Response
Have you been able to make use of these suggestions to improve my alignment algorithm?
Yes, if all plans views knew about some common vertical line in the model, and in each sheet view the position of that vertical line relative to the sheet, then that could be used to align views across sheet.
I’m not sure what that element would be? We also have the case were the two plan views have different cropping, and in fact may not have any common X,Y points – i.e., their crop boxes do not overlap. In general, views do not own elements that are outside their cropping.
The original algorithm modification using `View.Origin` with `View.Scale` has tested out to be reliable method to align plan views across sheets.
Interestingly, we are able to reliably create the 'mysterious offset' corrected for subtracting the scaled `View.Origin`.
The offset occurs when a plan view crop box is rotated. Here are the steps to reproduce:
- In the view properties, check 'Select crop region visible'.
- Select visible Crop Box.
- Rotate Crop Box using the Modify > Modify > Rotate command.
![Alignment of three planets over La Silla observatorium](img/Three_Planets_Dance_Over_La_Silla.jpg)