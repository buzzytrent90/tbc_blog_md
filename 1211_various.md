---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.518451'
original_url: https://thebuildingcoder.typepad.com/blog/1211_various.html
post_number: '1211'
reading_time_minutes: 5
series: general
slug: various
source_file: 1211_various.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- parameters
- references
- revit-api
- views
- walls
- windows
title: A Couple of Recent Issues
word_count: 963
---

### A Couple of Recent Issues

I have been quiet here for several days now, very busy working on Revit API cases, but nothing very generic to share here.

Instead, let me mention a couple of quick answers by Scott Conover of the Revit API development team to some other recent issues that cropped up:

- [Ribbon settings and drop down pin](#2)
- [Determining whether a family instance is joined](#3)
- [ReferenceIntersector and face](#4)
- [How to modify family instance geometry](#5)

#### Ribbon Settings and Drop Down Pin

**Question:** How can I access the ribbon drop down pin and settings arrow from the API?

![Ribbon pin](img/ribbon_pin.png)

![Ribbon settings arrow](img/ribbon_arrow.png)

**Answer:** The RibbonPanel AddSlideOut method adds a sliding panel to your custom add-in ribbon user interface.

You cannot do anything programmatically with the pin – the user has complete control over that.

If you want to do anything beyond that, you are leaving the realm of the officially supported Revit API and may be able to use the .NET
[UI Automation](http://thebuildingcoder.typepad.com/blog/automation) library
and the
[unsupported](http://thebuildingcoder.typepad.com/blog/about-the-author.html#4) functionality
provided by the AdWindows.dll assembly.

**Reponse:** Here is the code that I used to display the pin:

```csharp
  void PushButton\_Settings(
    RibbonPanel p)
  {
    p.AddSlideOut(); // this displays the pin!

    PushButtonData pbd = new PushButtonData(
      "Settings", "File path Settings",
      \_introLabPath, "HCL\_Ribbon.Command");

    pbd.LargeImage = NewBitmapImage("Setting-icon.png");
    pbd.Image = NewBitmapImage("Setting-icon.png");
    pbd.ToolTip = "Set the path to the folder location";

    p.AddItem( pbd );
  }
```
![Custom ribbon panel pin](img/ribbon_pin2.png)

#### Determining Whether a Family Instance is Joined

**Question:** Is there a way to tell if a family instance is joined to something?

I tried using RevitLookup but don't see anything useful.

When transforming an edge I'm getting different results after joining the family instance to another one or a wall, etc.

**Answer:** There are a couple of things to try:

- The JoinGeometryUtils GetJoinedElements method
- For concrete framing family instances and walls, you can retrieve the Element's Location, cast to LocationCurve, and look at the ElementsAtJoin collection.
- The Element GetGeneratingElementIds method tells you for a given piece of geometry from an element what element causes this geometry to form.
- Finally, you can use geometric analysis, e.g. via the ReferenceIntersector or a filter such as the ElementIntersectsSolidFilter

Here are some related discussions:

- [Joined beam geometry access](http://thebuildingcoder.typepad.com/blog/2011/01/joined-beam-geometry-access.html)
- [Filter for touching beams using solid intersection](http://thebuildingcoder.typepad.com/blog/2012/09/filter-for-touching-beams-using-solid-intersection.html)
- [Determining the columns supporting a beam](http://thebuildingcoder.typepad.com/blog/2013/03/supporting-columns.html#2)

The last link above discusses a whole bunch of different approaches.

#### Getting the Face from a ReferenceIntersector

**Question:** I am using the API to obtain a Face object at the intersection of a given point and direction.
For that, I used the ReferenceIntersector like this:

```csharp
  ReferenceIntersector refIntersector
    = new ReferenceIntersector( id,
      FindReferenceTarget.Face, selectedView );

  XYZ origin = new XYZ( x, y, z );

  ReferenceWithContext refContext
    = refIntersector.FindNearest(
      origin, new XYZ( 0, 0, -1 ) );

  Reference refObject = refContext.GetReference();
```

What next with the refObject?

How can I get a reference to the Face object?

**Answer:** From the reference, you can retrieve the associated element using the Document GetElement method.

From the element, you can retrieve the geometry object that was picked on it using GetGeometryObjectFromReference.

You can cast that to a face, which is not an element itself.

Many thanks to Scott and the team for raising and answering these questions.

#### How to Modify Family Instance Geometry

Oh, let me add one more issue that just came in right now.

**Question:** I am trying to modify the geometry contained in a FamilyInstance in a project document.
It does not appear that the API allows this.
I have figured out how to use the Duplicate method to create a new FamilySymbol in my project document and then assign it to the FamilyInstance.Symbol property.
But I can only seem to modify the parameter values for the duplicated symbol.
I really need to modify its geometry.
I would prefer to not have to go back into the family document to accomplish this because I am making these changes inside an Updater's Excute method.
Is there a way to accomplish this?
Please point me in the right direction.

**Answer:** Thank you for your query.

The short answer is 'no'.

I might add an emphatic 'absolutely not'.

Sorry.

You must be aware that the entire Revit model is parametrically driven.

The parameters drive the model.

The geometry is just the result.

To make any changes to the geometry, you have to start with the parameters.

They generate new geometry.

That is the main feature that makes
[Revit and its API different](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.41) from many other CAD systems:

I hope this helps.

This is really important to understand.

It helps a lot to get to know Revit well from the user interface, workflow, product side of things before starting to think about programming anything.

That helps convince you that the system actually works and does something useful, in addition to preventing you from doing everything you might be used to from programming other CAD systems.

By the way, the new
[DirectShape](http://thebuildingcoder.typepad.com/blog/2014/05/directshape-performance-and-minimum-size.html) element
does in fact enable you to define geometry right there in the project environment without creating a separate family and inserting an instance for it.

That was added to simplify IFC and other external format round trips, among other things, and does not change the main underlying principles, though.