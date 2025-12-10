---
post_number: "1733"
title: "Demolished Solid"
slug: "demolished_solid"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'geometry', 'parameters', 'references', 'revit-api', 'rooms', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1733_demolished_solid.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1733_demolished_solid.html"
---

### Determine Exact Opening by Demolishing
Here comes the most surprising Revit API functionality I have ever seen, put to a very useful and common task, determining the exact wall opening required for a door or window.
We have looked at numerous different approaches to determine wall openings in the past, including:
- [Opening geometry](http://thebuildingcoder.typepad.com/blog/2012/01/opening-geometry.html)
- [The temporary transaction trick for gross slab data](http://thebuildingcoder.typepad.com/blog/2012/10/the-temporary-transaction-trick-for-gross-slab-data.html)
- [Retrieving wall openings and sorting points](http://thebuildingcoder.typepad.com/blog/2015/12/retrieving-wall-openings-and-sorting-points.html)
- [Wall opening profiles](http://thebuildingcoder.typepad.com/blog/2015/12/wall-opening-profiles-and-happy-holidays.html#3)
- [Determining wall opening areas per room](http://thebuildingcoder.typepad.com/blog/2016/04/determining-wall-opening-areas-per-room.html#4)
- [More on wall opening areas per room](http://thebuildingcoder.typepad.com/blog/2016/04/more-on-wall-opening-areas-per-room.html)
- [Two energy model types](http://thebuildingcoder.typepad.com/blog/2017/01/family-category-and-two-energy-model-types.html#3)
- [IFC helper returns outer `CurveLoop` of door or window](https://thebuildingcoder.typepad.com/blog/2017/06/copy-local-false-and-ifc-utils-for-wall-openings.html#2)
So, it seems pretty hard to nail down, and pretty important to solve.
Now Håvard Leding of [Symetri](https://www.symetri.com) contributed
yet another exciting idea which highlights a number of surprising aspects,
demonstrates a further creative use case for `GetDependentElements` and expands on his
recent [RevitLookup enhancement to retrieve and snoop dependent elements](https://thebuildingcoder.typepad.com/blog/2019/03/retrieving-and-snooping-dependent-elements.html):
- [Get Demolished Solid](#2)
- [Why?](#3)
- [Questions?](#4)
In his own words:
#### Get Demolished Solid
Here is another use case for `GetDependentElements()`.
Determining the opening dimensions for Doors and Windows is surprisingly difficult, since you can't trust their parameters or reference planes to be consistent.
I suggest this alternative method that uses the solid that Revit creates when you demolish an opening.
It also uses the good
old [temporary transaction trick](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.53):
```csharp
///
/// If you demolish a door, Revit will automatically
/// fill the opening with a wall. So, we will use
/// that wall to get opening dimensions.
/// summary>
/// Door or Window is expectedparam>
private static Solid GetDemolishedSolid(
FamilyInstance fi )
{
Document doc = fi.Document;
Solid solidOpening = null;
using( Transaction t = new Transaction( doc ) )
{
if( fi.HasPhases() && fi.ArePhasesModifiable() )
{
t.Start( "temp" );
fi.DemolishedPhaseId = fi.CreatedPhaseId;
doc.Regenerate();
IList dependents
= fi.GetDependentElements( null );
foreach( ElementId id in dependents )
{
Element e = doc.GetElement( id );
if( e is Wall )
{
GeometryElement geomWall
= e.get_Geometry( new Options() );
// Need to clone it, as Rollback will
// destroy the original solid.
solidOpening = SolidUtils.Clone(
(Solid) geomWall.First() );
break;
}
}
t.RollBack();
}
}
// In the family editor, the host wall is always
// aligned ortho to the family coordinate system,
// so we can get the opening dimensions directly
// from the bounding box.
Solid solidInFamilyCoordinates
= SolidUtils.CreateTransformed( solidOpening,
fi.GetTotalTransform().Inverse );
BoundingBoxXYZ bb = solidInFamilyCoordinates.GetBoundingBox();
XYZ dimensions = bb.Max - bb.Min;
TaskDialog.Show( "Opening", "Opening is "
+ dimensions.X.ToString() + " by "
+ dimensions.Z.ToString() );
return solidOpening;
}
```
#### Why?
I agree some more explanation as to why parameters and refplanes are not to be trusted would be nice.
You can't rely on any built-in parameters being in use.
Or used as you expect them to be used.
Same goes for reference planes.
Sometimes, it's just static geometry. (No parameters in use)
And sometimes, it's just an opening family. (No solids)
Still, you need to analyse something, and preferably solids.
But sometimes, solids don't represent the true opening, like in this case, where you have a gap:
![Window opening with gap](img/window_opening_with_gap.jpg)
It's just safer and a lot easier to use the final 'opening solid' from a demolished state.
If you want to try it out yourself, the method is easy enough to use;
it just requires a door or window to run :-)
#### Questions?
\*\*Question:\*\* Reading the code in more detail, though, I don't really understand...
The family instance `fi` is a window, for example, yes?
From the window, you call `GetDependentElements`, which returns "all elements that, from a logical point of view, are the children of this Element".
From those, you extract the wall.
Hmm. So, the wall is dependent on the window? That seems weird.
OK, let's accept that the wall is returned.
Now, from the wall, you retrieve the first geometry element.
That is now saved as the solid opening.
That seems super weird. I would have thought that the wall geometry is the wall geometry, and that the hole is a hole.
Why is the first solid in the wall geometry a solid representing the opening?
Does that really work?
Can you explain?
\*\*Answer:\*\* Yes, it works; just try it :-)
It works, because when you demolish openings, Revit automatically fills the entire hole with a new Wall.
You can see it if you manually demolish a window.
There is no longer a hole in the wall.
Very easy to get dimensions from the bounding box, once transformed into the family editor coordinate system.
Because there, the host wall direction is always aligned with the coordinate system cardinal axes.
I guess opening dimensions could be extracted without the transform, but I wouldn't know how to do that.
You could also analyse the shape of the opening by looking at the vertical front face of the solid.
You would have to do so to handle cases where the window is not rectangular, e.g., circular or something else.
\*\*Response:\*\* Wow. I am impressed. And surprised.
Many thanks to Håvard for this innovative solution and interesting explanation!