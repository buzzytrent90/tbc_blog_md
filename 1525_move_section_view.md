---
post_number: "1525"
title: "Move Section View"
slug: "move_section_view"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'revit-api', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1525_move_section_view.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1525_move_section_view.html"
---

### Moving the Section View Location
Today, let's recap
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [moving the location of a section view](http://forums.autodesk.com/t5/revit-api-forum/move-location-of-section-view/m-p/6831743) raised
by [Danny Bentley](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/2615791),
BIM Structural Technician at [SOM](http://www.som.com) in California, since Danny very kindly created a video and GitHub repo to demonstrate and share the solution, which will certainly be of use to others as well.
![Danny Bentley](img/danny_bentley_x120.jpg)
By the way, Danny also
writes [Bentley's Revit Dynamo & API blog](http://revitdynamoapi.blogspot.ch/) on
his personal exploration of the Revit API and Dynamo.
Why does it have a `ch` domain, Danny?
Anyway, here is the issue moving the location of a section view:
\*\*Question:\*\* I saw a great post on the \*The Building Coder\* on how
to [create a section view to be aligned with a wall](http://thebuildingcoder.typepad.com/blog/2012/06/create-section-view-parallel-to-wall.html).
I'm working on a project in which the wall angle will change and I need to rotate an existing section to match it.
Is it possible to set a curve similar to a wall? I can't seem to find any curve or location point on the section view in RevitLookup.
```csharp
LocationCurve locationCurve = wall.WallCurve;
Transaction t = new Transaction( doc );
t.Start( "Move Wall" );
locationCurve.Curve = LinkedProfile;
t.Commit();
```
\*\*Answer:\*\* Thank you for your query and appreciation.
First of all, out of habit, I repeat:
For your own comfort and security,
[all use of transactions should be encapsulated in a `using` statement](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html).
The Building Coder defines an entire topic group
on [handling transactions and transaction groups](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.53).
I think you can indeed change the wall location curve in a manner similar to what you show.
However, the code you list above is guaranteed not to work.
Beginning of misunderstanding
Look:
In this line, you make a copy of the wall location curve:
```csharp
LocationCurve locationCurve = wall.WallCurve;
```
Then, you change something in the copy:
```csharp
locationCurve.Curve = LinkedProfile;
```
And then you expect the wall to change?
Good luck with that.
You might have better luck with
```csharp
wall.WallCurve = LinkedProfile;
```
You might also want to take a look at
the [ADN Xtra labs](https://github.com/jeremytammik/AdnRevitApiLabsXtra) sample
command [Lab2_5_SelectAndMoveWallAndAddColumns](https://github.com/jeremytammik/AdnRevitApiLabsXtra/blob/master/XtraCs/Labs2.cs#L837-L1102).
It moves a wall using
```csharp
wall.Location.Move( v )
```
by a given vector v.
End of misunderstanding
Oh, no, re-reading your query, I see you are trying to modify the section view, not the wall location line.
No, the section view does not have a location line.
It is controlled differently. The Revit SDK sample DynamicModelUpdate shows how to dynamically update a section view to follow a window placed in the model. You can adapt that to follow a wall in a similar fashion.
Here is a quick rundown of the places to look in that sample step by step:
- Folder `2017.1/SDK/Samples/DynamicModelUpdate/CS`
- Module `SectionUpdater.cs`
- Method `public void Execute(UpdaterData data)`
- Variables and method call:
```csharp
FamilyInstance window = doc.GetElement( m_windowId ) as FamilyInstance;
ViewSection section = doc.GetElement( m_sectionId ) as ViewSection;
RejustSectionView( doc, window, section );
```
- Method implementation and action:
```csharp
internal void RejustSectionView( Document doc,
Element elem, ViewSection section )
ElementTransformUtils.RotateElement(doc, m_sectionElement.Id,
axis, rotateAngle);
ElementTransformUtils.MoveElement(doc, m_sectionElement.Id,
translationVec);
```
\*\*Response:\*\* Perfect! I forget the many awesome examples in the SDK. I really need to look through them instead of always using Google.
\*\*Answer:\*\* I think you need to use both SDK and Internet searches, and more besides, as described
in [how to research to find a Revit API solution](http://thebuildingcoder.typepad.com/blog/2017/01/virtues-of-reproduction-research-mep-settings-ontology.html#3).
\*\*Response:\*\* I just wanted to add my solution in case anyone else is looking to align section views and create aligned sections.
Here is a [30-second video on YouTube](https://www.youtube.com/watch?v=gkgV2Ff6zC8) showing my add-in in action:

The source code is provided on GitHub, in [dannysbentley Sections repo](https://github.com/dannysbentley/Sections).
It sports four external commands:
- CommandUpdate
- CommandCreateStart
- CommandCreateEnd
- CommandCreatePerpendicular
It also creates a neat little external application presenting a custom ribbon panel to drive them.
Many thanks to Danny for sharing this solution!