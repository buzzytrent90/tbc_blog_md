---
post_number: "1680"
title: "Refplane In Family"
slug: "refplane_in_family"
author: "Jeremy Tammik"
tags: ['family', 'geometry', 'references', 'revit-api', 'sheets', 'transactions', 'views']
source_file: "1680_refplane_in_family.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1680_refplane_in_family.html"
---

### Icon Access and Reference Plane in Family Instance
Today I highlight two of Fair59's numerous solutions in the Revit API forum, and also mention a joke I picked up in a Swedish cartoon:
- [Fadaesen on racism versus realism](#2)
- [Retrieving a reference plane location in a family instance](#3)
- [Accessing the Revit ribbon icons](#4)
#### Fadaesen on Racism versus Realism
As I [recently mentioned](http://thebuildingcoder.typepad.com/blog/2018/08/revit-20191-cefsharp-forge-accelerator-in-rome.html),
I visited Sweden in the beginning of August for a week's hiking.
Before getting to the Revit API stuff, let me share this little cartoon that I enjoyed in the train there.
The artist and author, [Marja Nyberg](http://www.marjaproductions.se), very kindly gave me permission to translate and reproduce it here:
![Fadaesen on racism](img/2018-08-14_marja_nyberg_fadaesen.jpg)
- i am not a racist, i'm just being realistic
- OK, OK i am not a human being, i am a miniature horse
- what do you mean? of course you're not
- oh sorry, i thought we were playing the game 'negate what you obviously are and then say something random as far as possible removed from the truth'
- hey, you just listen now! i am not an idiot! don't you think i can see what you and your likes are up to in this country?
- ... are you quite sure we are not playing that game? because that last one was REALLY nice
Back to the Revit API, though...
#### Retrieving a Reference Plane Location in a Family Instance
Fair59 continues providing pretty advanced, tricky and just-about perfect solutions to many challenging questions in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
such as for this thread
on [getting the position of a reference plane in a family instance](https://forums.autodesk.com/t5/revit-api-forum/get-position-referenceplane-in-familyinstance/m-p/8243347):
\*\*Question:\*\* Is it possible to determine the position of a reference plane in a family instance that is not aligned with any mass or other object?
I need its direction and position.
I tried to use the `ReferenceIntersector`, but it does not detect this reference.
\*\*Answer:\*\* You could start out from the very informative and useful previous thread on retrieving
the [direction of a reference (reference plane or reference line)](https://forums.autodesk.com/t5/revit-api-forum/direction-of-reference-reference-plane-or-reference-line/m-p/7074163).
That only determines the direction.
However, from the dimension in the code, you could also find a point on the plane.
On the other side, here is an alternative approach:
You can create a sketch plane using the reference, then query the origin and normal from the geometry plane of the sketch plane.
```csharp
///
/// Retrieve origin and direction of the left
/// reference plane within the given family instance.
/// summary>
static bool GetFamilyInstanceReferencePlaneLocation(
FamilyInstance fi,
out XYZ origin,
out XYZ normal )
{
bool found = false;
origin = XYZ.Zero;
normal = XYZ.Zero;
Reference r = fi
.GetReferences( FamilyInstanceReferenceType.Left )
.FirstOrDefault();
if( null != r )
{
Document doc = fi.Document;
using( Transaction t = new Transaction( doc ) )
{
t.Start( "Create Temporary Sketch Plane" );
SketchPlane sk = SketchPlane.Create( doc, r );
if( null != sk )
{
Plane pl = sk.GetPlane();
origin = pl.Origin;
normal = pl.Normal;
found = true;
}
t.RollBack();
}
}
return found;
}
```
Many thanks to Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen
for this effective solution, and his many other helpful answers, and also
to Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) Ignatovich, aka Александр Игнатович, for the blog post suggestion.
By the way, I added this method
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples),
which heretofore lacked any example at all of using the relatively new family instance `GetReferences` method, in
the [module `CmdDimensionInstanceOrigin.cs`](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdDimensionInstanceOrigin.cs).
#### Accessing the Revit Ribbon Icons
Fair59 and Matt Taylor also recently provided another useful forum answer, in the two threads
on [Revit ribbon icons and pictograms](https://forums.autodesk.com/t5/revit-api-forum/revit-ribbon-icons-pictograms/m-p/8126270)
and [view icons in Revit](https://forums.autodesk.com/t5/revit-api-forum/view-icons-in-revit/m-p/7156748):
\*\*Question:\*\* I would like to display icons instead of dumb text in my add-in ribbon user interface.
Does anybody where I can find the built-in Revit icons and pictograms?
\*\*Answer:\*\* You can quickly check to see what icons are available in any DLL by:
- Right click on any desktop shortcut > Properties > Change Icon > Browse to the DLL file.
The built-in Revit icons are contained in the file `Utility.dll` in the same folder as Revit.exe.
Many thanks to Frank and Matt for helping to solve these two questions as well.