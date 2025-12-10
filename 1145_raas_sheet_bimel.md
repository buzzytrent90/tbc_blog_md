---
post_number: "1145"
title: "Revit as a Service and Sheet-View-Element Transforms"
slug: "raas_sheet_bimel"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'references', 'revit-api', 'rooms', 'selection', 'sheets', 'views', 'walls', 'windows']
source_file: "1145_raas_sheet_bimel.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1145_raas_sheet_bimel.html"
---

### Revit as a Service and Sheet-View-Element Transforms

Several people recently brought up the question of running Revit as a service, e.g. in this
[Revit API forum discussion thread](http://forums.autodesk.com/t5/Revit-API/Revit-server-API-access/m-p/4971680),
so let us take a closer look at that.

I also finally got around to making some seriously overdue progress on the RoomEditorApp, which I should really rename to SimplifiedBimEditorApp.
It now displays a Windows preview form showing the sheets, the views, and the elements they contain in preparation to uploading them to a cloud database.

While I'm at it, I'll also add an explanation on the relationship – or non-relationship – between line style and line pattern.

So here goes:

- [On running Revit as a service](#2)
- [Sheet view element display transformation](#3)
- [Non-relationship between Line Style and Line Pattern](#4)

#### On Running Revit as a Service

I already presented examples of running Revit as a service by making use of the Idling or an external event, e.g. to
[drive Revit through a WCF service](http://thebuildingcoder.typepad.com/blog/2012/11/drive-revit-through-a-wcf-service.html),
proving it at least technically feasible.

One important question in this context is the legality of making serious use of this possibility.

**Question:** I have implemented a Revit add-in that uses our existing stand-alone program to generate geometry that is fed into the Revit API for creating windows and doors in the family editor.
This works nicely on the desktop version of Revit, and we have implemented a similar SketchUp plugin too.

I would like to allow online users to use the Revit API to generate their families and types as well.
The release of Revit LT makes this even more important, since it does not allow the use of add-ins.
If we could run Revit as a service, it would also allow us to have a presence on
[Autodesk Seek](http://seek.autodesk.com).

So we would like to run Revit as a service, send online-collected parameters via a web service, and asynchronously return a Revit RFA file to the requestor.

Ideally, we just need the Revit family editor to be running as a service.

Is that finally possible with Revit 2015? Or Autodesk 360?

**Answer:** The Revit and other Autodesk software licenses do not officially allow running the software as a service.

On the other hand, technically it definitely is possible.

As Adam explained in the
[forum thread](http://forums.autodesk.com/t5/Revit-API/Revit-server-API-access/m-p/4971680),
"no headless or server version of Revit is available.
Technically it should be possible to have a Revit on a server that is accessed from other computers to do some processing.
I do not know the legal side of it though – e.g. if all users accessing this server would need to have a full Revit license or not, or if even that would not make it legal."

Here is an add-in showing how you can
[drive Revit through a WCF service](http://thebuildingcoder.typepad.com/blog/2012/11/drive-revit-through-a-wcf-service.html).

The Building Coder also provides a number of other examples of
[driving Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28).

There are licensing issues to overcome and you may want to talk to the appropriate people at Autodesk about using a Revit in this way.

It is legal to run Revit on any machine (user or server) as long as there is a valid licence available for it. Running regular Revit on a server would require depending on the Idling or an external ß event mechanism.

It is not legal and hopefully not possible either to run external applications with Revit LT.

You can also discuss with Autodesk Consulting about building custom solutions making use of Revit as a service.

#### Sheet View Element Display Transformation

The last update on my Tech Summit preparations showed how to
[determine the size and location of viewports on a sheet](http://thebuildingcoder.typepad.com/blog/2014/04/determining-the-size-and-location-of-viewports-on-a-sheet.html).

Retrieving the geometry of the elements displayed in the views and transforming them appropriately to fit inside the viewports displayed on a sheet is a bit more complex.
It required some struggles and refactoring of the RoomEditorApp code.

Happily I managed, however.

Here is a simple sample sheet with three views:

![Sheet with three views](img/sheet_bimel_01.png)

I implemented the code to display an simplified view of this in a temporary Windows .NET form generated on the fly in preparation to uploading it to a NoSQL cloud database.
The required steps include the calculations of the following transformation to:

- Scale the sheet to the form
- Place each view in its corresponding viewport on the sheet
- Place the elements displayed within each view in the appropriate positions in each viewport
- Handle family instance transformations

The resulting form looks like this:

![Windows preview of sheet, views and BIM elements](img/sheet_bimel_02.png)

All elements from views that are not floor plans are ignored.

I determine the element scaling by rather naively calculating the bounding box of all elements within each view and mapping that to the viewport rectangle.

That is obviously not precise.
However, it should suffice for my needs.

As explained when discussing the
[selection of visible categories from a set of views](http://thebuildingcoder.typepad.com/blog/2014/03/selecting-visible-categories-from-a-set-of-views.html),
only specific BIM elements belonging to selected categories are retrieved and displayed.

Obviously, we have no interest in showing the elevation view markers and such-like stuff.

Curtain walls are currently missing, because I am not handling the way their geometry is defined.

The code to achieve this is provided in full by the updated RoomEditorApp, Visual Studio solution and add-in manifest in the
[RoomEditorApp GitHub repository](https://github.com/jeremytammik/RoomEditorApp).

The version discussed above is
[release 2015.0.2.10](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2015.0.2.10).

After finishing and publishing the above, I cleaned up the code a bit and added one more small but significant feature.

Previously, I was simply ignoring family instances with no location property.

That was causing the curtain walls to disappear.

I modified the code to treat family instances with no location point as fixed building element parts instead.

Lo and behold, the curtain walls now appear just fine:

![Preview including curtain walls](img/sheet_bimel_03.png)

The updated version fresh off the press is
[release 2015.0.2.11](https://github.com/jeremytammik/RoomEditorApp/releases/tag/2015.0.2.11).

#### Non-relationship Between Line Style and Line Pattern

The following is a summary of the Revit API discussion forum thread on
[how to get the relationship between a line style and a line pattern](http://forums.autodesk.com/t5/Revit-API/How-to-get-the-relationship-between-a-Line-Style-and-a-Line/m-p/4993274):

**Question:** I am struggling to figure out how to find the relationship between a line style and a line pattern in the Revit project environment.

For example, line styles are basically a subcategory of the Line category. The general Line category can have many Line Style subcategories, and all subcategories of the Line category apparently \*are\* line styles.

To get to line patterns, you use a FilteredElementCollector.OfClass (LinePatternElement).

Line Patterns have a name and a list of LineSegment classes, each which has a Length double and an enum LineSegmentType (e.g. dash, dot or space).

Each line style can have exactly 1 line pattern or no line patterns. What we need to find out is which line pattern (if any) is related to each line style (subcategory). There does not seem to be, for example, a LinePattern property on the Category class (Category class is used for both Categories and SubCategories, so in reality a Line Style is of the Category class).

**Answer:** This was discussed not long ago in the following Q & A:

**[Q]** The Revit documentation says that Line Patterns are used in the definition of GraphicsStyle objects: "A line pattern is a pattern of dashes and dots used to control the way the lines of an object are drawn in Revit. Line patterns are used in the definition of GraphicsStyle objects. A line pattern is defined by a repeating sequence segments. Each segment is a dash, a dot or a space. A line pattern definition must contain an even number of segments, starting with a visible segment (a dash or a dot) and alternating between visible segments and spaces." However, I could not find any APIs to connect these objects. How are these objects connected and how to use the APIs to connect them?

**[A]** Revit does currently not expose the API to set or get a GraphicsStyle object's line pattern element. Users can only change/get the line pattern for GraphicsStyle object via the Revit UI. You can create a new line pattern element in Revit 2014 using its constructor to create a LinePattern object: LinePattern(String). You can call the LinePattern.SetSegment() method to create customized line patterns. You can create a LinePatternElement via LinePatternElement.Create(LinePattern).

Based on that, I don't think we currently expose any link between GraphicsStyle and line pattern. I don't think we even have the ability to get a line pattern directly from the category.

However, you can get the pattern id for a specific view via two OverrideGraphicSettings properties: ProjectionLinePatternId and CutLinePatternId. These get the value from the Visibility/Graphics dialog, not object styles. Not sure if it will return the object styles value if it's not overridden in the view.

Clearing up a few incorrect assumptions in the query itself:

- Line style != subcategory or category. It's not a 1:1 relationship.
- Each category and subcategory has two GraphicsStyles: cut and projection (selected via GraphicsStyleType). Each style has a color, weight & pattern reference.
- Patterns are shared by multiple styles, so you can't map back from a line pattern back to a single line style or category because it's a one to many relationship.