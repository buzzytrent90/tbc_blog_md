---
post_number: "1679"
title: "Delete Unused Ref Plane"
slug: "delete_unused_ref_plane"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'parameters', 'references', 'revit-api', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1679_delete_unused_ref_plane.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1679_delete_unused_ref_plane.html"
---

### Reference Intersector and Deleting Reference Planes
We already looked
at [deleting unnamed non-hosting reference planes](http://thebuildingcoder.typepad.com/blog/2014/02/deleting-unnamed-non-hosting-reference-planes.html) back
in 2012 and 2014.
Some things have changed since then, and the old code requires fixing and updating once again.
Other interesting topics also want to be mentioned:
- [Embodyment workshop](#2)
- [Bös Fulen mountain hike](#3)
- [Using `ReferenceIntersector` to place lighting fixture on ceiling face](#4)
- [Reformat stable representation string for dimensioning](#5)
- [Deleting unnamed non-hosting reference planes updated](#6)
#### Embodyment Workshop
I attended a very nourishing dance workshop
with [Alain Allard](https://www.facebook.com/pg/Alain-Allard-5-Rhythms-Moves-into-Consciousness-179038505454123)
of [Moves into Consciousness](http://www.movesintoconsciousness.com), UK.
Alain is a dancer, fully licensed and practising psychotherapist, and meditates.
On that basis, he led us through two very fulfilling days of attending to the rhythms of the body and listening more clearly the rhythms of life, the voice of the heart and gut intelligence, practicing giving up the everyday habits of dulling our creative response to Life’s unfolding by numbing down and overthinking.
It was very enjoyable and enlivening indeed!
Thank you, Alain, and thank you, Bea, for inviting and organising!
#### Bös Fulen Mountain Hike
I followed up on the dance workshop with a mountain hike with Alex to
the [Bös Fulen](https://en.wikipedia.org/wiki/B%C3%B6s_Fulen),
highest mountain in the Swiss Canton Schwyz:
![Bös Fulen](img/boes_fulen.png)
In spite of its looks, it in in fact a pretty easy hike across the entire ridge a couple of hundred meters to the summit.
Unfortunately, the weather we had was not as good as in the picture above.
We had some sight, though much of it was of dramatic clouds forming all around us, and we got pretty wet walking out again.
Here is Alex' [15-minute GoPro video of the foggy walk back across the ridge](https://youtu.be/jPsrhhfrUGs), just before the rain started coming in fully:

#### Using ReferenceIntersector to Place Lighting Fixture on Ceiling Face
Returning to the Revit API, Joshua Lumley shared a nice little example of using the `ReferenceIntersector` class to determine a point on and a reference to the ceiling face in a linked file to place a lighting fixture above a given point, in
his [comment](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html#comment-4067218487) on the old discussion of
the [`FindReferencesByDirection` method](http://thebuildingcoder.typepad.com/blog/2010/01/findreferencesbydirection.html), the precursor of `ReferenceIntersector`.
I generalised and simplified his code snippet to this `GetCeilingReferenceAbove` method, which I added
to [The Building Coder Samples](https://github.com/jeremytammik/the_building_coder_samples)
in the module `CmdDimensionWallsFindRefs.cs`:
```csharp
///
/// Return reference to ceiling face to place
/// lighting fixture above a given point.
/// summary>
Reference GetCeilingReferenceAbove(
View3D view,
XYZ p )
{
ElementClassFilter filter = new ElementClassFilter(
typeof( Ceiling ) );
ReferenceIntersector refIntersector
= new ReferenceIntersector( filter,
FindReferenceTarget.Face, view );
refIntersector.FindReferencesInRevitLinks = true;
ReferenceWithContext rwc = refIntersector.FindNearest(
p, XYZ.BasisZ );
Reference r = (null == rwc)
? null
: rwc.GetReference();
if( null == r )
{
System.Windows.MessageBox.Show( "no intersecting geometry" );
}
return r;
}
```
Here is a sample snippet showing how the method expects to be used:
```csharp
void TestGetCeilingReferenceAbove( Document doc )
{
View3D view = doc.GetElement( new ElementId( 147335 ) ) as View3D;
Space space = doc.GetElement( new ElementId( 151759 ) ) as Space;
XYZ center = ( (LocationPoint) space.Location ).Point;
Reference r = GetCeilingReferenceAbove( view, center );
// Populate these as needed:
XYZ startPoint = null;
FamilySymbol sym = null;
doc.Create.NewFamilyInstance( r, startPoint, XYZ.BasisY, sym );
}
```
#### Reformat Stable Representation String for Dimensioning
By the way, Joshua also added an important note on how
to [reformat the stable representation string for dimensioning](http://thebuildingcoder.typepad.com/blog/2016/04/stable-reference-string-magic-voodoo.html#6) in the discussion
on the [reference stable representation magic voodoo](http://thebuildingcoder.typepad.com/blog/2016/04/stable-reference-string-magic-voodoo.html).
Many thanks for your helpful contributions, Joshua!
#### Deleting Unnamed Non-Hosting Reference Planes Updated
Another important update was initiated by Austin Sudtelgte, who pointed out that the approach described in 2014
for [deleting unnamed non-hosting reference planes](http://thebuildingcoder.typepad.com/blog/2014/02/deleting-unnamed-non-hosting-reference-planes.html) no longer works.
[Austin adds](http://thebuildingcoder.typepad.com/blog/2014/02/deleting-unnamed-non-hosting-reference-planes.html#comment-3985629186):
> As of 2017 this method no longer works, because Revit doesn't throw an error when deleting a plane with something hosted to it. You will have to get all family instances in the document, check their host ID parameter, get all reference planes excluding the ones whose IDs we just collected, then delete. Dimensions that measure to a reference plane will be removed with the reference plane. The methods used above will still work to determine if one of those will be deleted or not. Alternatively, as of 2018.1, there is an `Element.GetDependentElements` method that will return the same things as what is returned when deleting the element, just without having to delete it and roll back a transaction.
He also points out in
his [comment on \*What's New in the Revit 23019 API\*](http://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html#comment-3985568426):
> [Point 2.3 on finding element dependencies](http://thebuildingcoder.typepad.com/blog/2018/04/whats-new-in-the-revit-2019-api.html#4.2.3) with
`Element.GetDependentElements` seems to require an element filter rather than having it be optional as noted above. It's possible that in 2019 the filter is optional, but in 2018.1 it was not.
This is relevant
in [The Building Coder sample](https://github.com/jeremytammik/the_building_coder_samples)
`CmdDeleteUnusedRefPlanes.cs` to delete unnamed non-hosting reference planes, originally discussed and implemented at
the [Melbourne DevLab](http://thebuildingcoder.typepad.com/blog/2012/03/melbourne-devlab.html#2) in 2012 to delete all reference planes that:
- Have not been assigned a name, and
- Do not host any elements.
The original approach stopped working and was [updated and added to The Building Coder samples in 2014](http://thebuildingcoder.typepad.com/blog/2014/02/deleting-unnamed-non-hosting-reference-planes.html),
when the `Delete` method overload taking an `Element` argument was deprecated in Revit 2014 and the deletion of a reference plane hosting elements started throwing an exception.
Austin shared his version of the broken command, plus his new working solution.
I added them both
to [The Building Coder Samples](https://github.com/jeremytammik/the_building_coder_samples)
[module `CmdDeleteUnusedRefPlanes.cs`](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdDeleteUnusedRefPlanes.cs).
Yet again, I discovered a couple of optimisation possibilities that I very frequently keep pointing out and also added to Austin's code:
- Iterate directly over the filtered element collectors instead of converting it to a list, which wastes time and space for no benefit whatsoever.
- Use a `Dictionary` instead of a `List`, since it is much more efficient looking up a dictionary key than searching an unsorted list for a specific entry.
- Only instantiate the transaction if it is actually going to be used.
You can see exactly what I modified in
the [commit highlighting those changes](https://github.com/jeremytammik/the_building_coder_samples/commit/21ef175572ee1b4ac7933af391135004c8889343).
Here Austin's solution in its current form:
```csharp
[TransactionAttribute( TransactionMode.Manual )]
public class CmdDeleteUnusedRefPlanes : IExternalCommand
{
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiapp = commandData.Application;
UIDocument uidoc = uiapp.ActiveUIDocument;
Document doc = uidoc.Document;
ICollection refplaneids
= new FilteredElementCollector( doc )
.OfClass( typeof( ReferencePlane ) )
.ToElementIds();
using( TransactionGroup tg = new TransactionGroup( doc ) )
{
tg.Start( "Remove unused reference planes" );
FilteredElementCollector instances
= new FilteredElementCollector( doc )
.OfClass( typeof( FamilyInstance ) );
Dictionary toKeep
= new Dictionary();
foreach( FamilyInstance i in instances )
{
// Ensure the element is hosted
if( null != i.Host )
{
ElementId hostId = i.Host.Id;
// Check list to see if we've already added this plane
if( !toKeep.ContainsKey( hostId ) )
{
toKeep.Add( hostId, 0 );
}
++toKeep[hostId];
}
}
// Loop through reference planes and
// delete the ones not in the list toKeep.
foreach( ElementId refid in refplaneids )
{
if( !toKeep.ContainsKey( refid ) )
{
using( Transaction t = new Transaction( doc ) )
{
t.Start( "Removing plane "
+ doc.GetElement( refid ).Name );
// Ensure there are no dimensions measuring to the plane
if( doc.Delete( refid ).Count > 1 )
{
t.Dispose();
}
else
{
t.Commit();
}
}
}
}
tg.Assimilate();
}
return Result.Succeeded;
}
}
```
Many thanks to Austin for picking up and fixing this!