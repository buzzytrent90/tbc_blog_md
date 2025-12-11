---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:16.570048'
original_url: https://thebuildingcoder.typepad.com/blog/1707_filter_intersect.html
post_number: '1707'
reading_time_minutes: 5
series: filtering
slug: filter_intersect
source_file: 1707_filter_intersect.md
tags:
- elements
- family
- filtering
- geometry
- revit-api
- selection
- sheets
- views
- walls
title: Filter Intersect
word_count: 1039
---

### Using an Intersection Filter for Linked Elements
Intersecting elements has always been a hot topic, cf. various previous discussions
on [3D Booleans, cutting and joining elements](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.30);
intersecting with elements in a linked file is even more challenging.
Happily, the Revit API provides tools to support that as well:
- [Intersecting linked elements with current project ones](#2)
- [Retrieving rebars intersecting a structural element](#3)
![Rebar intersecting column](img/rebar_intersect_column.png)
#### Intersecting Linked Elements with Current Project Ones
Yongyu [@wlmsingle](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/6363417) Deng raised and answered an interesting question in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to use the `ElementIntersectsElementFilter` from the `RevitLinkInstance`](https://forums.autodesk.com/t5/revit-api-forum/how-to-use-the-elementintersectselementfilter-from-the/m-p/8440333) to
retrieve MEP elements from a linked file and intersect them with structural elements in the current project:
\*\*Question:\*\* I need to get the MEP elements from a linked file and intersect them with some structural elements in the current project.
I succeeded with this using the `ElementIntersectsElementFilter` and the `ElementIntersectsSolidFilter`.
The filters are both works fine if there is no transform (i.e., identity) between the `RevitLinkInstance` and the current project.
However, if the `RevitLinkInstance` is moved or rotated after importing, the calculation result of the filters is incorrect.
Are there any tricks to solve a case like that?
For example, a method for passing a transform to the filter?
If not, please share a good algorithm to get the intersection result between two solid elements.
\*\*Answer:\*\* Retrieve solids from the elements of interest and use
the [`SolidUtils.CreateTransformed` method](https://apidocs.co/apps/revit/2019/22592761-f39c-4f53-d33b-6c21a4fa9d2d.htm) on them.
\*\*Response:\*\* Thanks a lot!
You inspired me to develop a new solution.
If there is a non-identity transform on the `RevitLinkInstance`, the key point to use the `ElementIntersectsSolidFilter` correctly is simply to transform the element in the current project to the linked file project coordinate system:
Here is a slightly cleaned up version of Yongyu Deng's code that I added to
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2019.0.144.4](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.144.4):
```csharp
///
/// Collect the element ids of all elements in the
/// linked documents intersecting the given element.
/// summary>
/// Target elementparam>
/// Linked documentsparam>
/// Return intersecting element idsparam>
/// Number of intersecting elements foundreturns>
int GetIntersectingLinkedElementIds(
Element e,
IList links,
List ids )
{
int count = ids.Count();
Solid solid = GetSolid( e );
foreach( RevitLinkInstance i in links )
{
// GetTransform or GetTotalTransform or what?
Transform transform = i.GetTransform();
if( !transform.AlmostEqual( Transform.Identity) )
{
solid = SolidUtils.CreateTransformed(
solid, transform.Inverse );
}
ElementIntersectsSolidFilter filter
= new ElementIntersectsSolidFilter( solid );
FilteredElementCollector intersecting
= new FilteredElementCollector( i.GetLinkDocument() )
.WherePasses( filter );
ids.AddRange( intersecting.ToElementIds() );
}
return ids.Count - count;
}
```
#### Retrieving Rebars Intersecting a Structural Element
Another [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
deals with [getting all associated rebars that attach to a structural element](https://forums.autodesk.com/t5/revit-api-forum/get-all-associated-rebars-which-attach-to-the-structural-element/m-p/8446328):
\*\*Question:\*\* How can I get all associated rebars which attach to a structural element such as a column by picking that?
\*\*Answer:\*\* Picking an element is described in
the [Revit API getting started material](https://thebuildingcoder.typepad.com/blog/about-the-author.html#2) and
also demonstrated in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).
For instance, in the latter, you can check out
the [various element selection utility methods](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/Util.cs#L1227-L1365) and
examine how they are used in the sample commands.
Once you have found a usage pattern that you like in some sample command, search for the description of it
in [The Building Coder blog](https://thebuildingcoder.typepad.com).
Once you have picked your structural element, use a filtered element collector to retrieve the intersecting rebar.
Set it up to retrieve rebar elements only, and add a filter for the column solid:
- [Bounding box filter is always axis aligned](https://thebuildingcoder.typepad.com/blog/2018/04/bounding-box-filter-always-axis-aligned.html)
- [Using intersection filter with linked file](https://thebuildingcoder.typepad.com/blog/2018/04/using-intersection-filter-with-linked-file.html)
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
includes some examples of using a solid intersection filter, e.g., the [`GetInstancesIntersectingElement` method](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs#L1294-L1430) showing
how to retrieve family instances intersecting a given BIM element:
```csharp
///
/// Retrieve all family instances intersecting a
/// given BIM element, e.g. all columns
/// intersecting a wall.
/// summary>
void GetInstancesIntersectingElement( Element e )
{
Document doc = e.Document;
Solid solid = e.get_Geometry( new Options() )
.OfType()
.Where( s => null != s && !s.Edges.IsEmpty )
.FirstOrDefault();
FilteredElementCollector intersectingInstances
= new FilteredElementCollector( doc )
.OfClass( typeof( FamilyInstance ) )
.WherePasses( new ElementIntersectsSolidFilter(
solid ) );
int n1 = intersectingInstances.Count();
intersectingInstances
= new FilteredElementCollector( doc )
.OfClass( typeof( FamilyInstance ) )
.WherePasses( new ElementIntersectsElementFilter(
e ) );
int n = intersectingInstances.Count();
Debug.Assert( n.Equals( n1 ),
"expected solid intersection to equal element intersection" );
string result = string.Format(
"{0} family instance{1} intersect{2} the "
+ "selected element {3}{4}",
n, Util.PluralSuffix( n ),
( 1 == n ? "s" : "" ),
Util.ElementDescription( e ),
Util.DotOrColon( n ) );
string id_list = 0 == n
? string.Empty
: string.Join( ", ",
intersectingInstances
.Select(
x => x.Id.IntegerValue.ToString() ) )
+ ".";
Util.InfoMsg2( result, id_list );
}
///
/// Retrieve all beam family instances
/// intersecting two columns, cf.
/// http://forums.autodesk.com/t5/revit-api/check-to-see-if-beam-exists/m-p/6223562
/// summary>
FilteredElementCollector
GetBeamsIntersectingTwoColumns(
Element column1,
Element column2 )
{
Document doc = column1.Document;
if( column2.Document.GetHashCode() != doc.GetHashCode() )
{
throw new ArgumentException(
"Expected two columns from same document." );
}
FilteredElementCollector intersectingStructuralFramingElements
= new FilteredElementCollector( doc )
.OfClass( typeof( FamilyInstance ) )
.OfCategory( BuiltInCategory.OST_StructuralFraming )
.WherePasses( new ElementIntersectsElementFilter( column1 ) )
.WherePasses( new ElementIntersectsElementFilter( column2 ) );
int n = intersectingStructuralFramingElements.Count();
string result = string.Format(
"{0} structural framing family instance{1} "
+ "intersect{2} the two beams{3}",
n, Util.PluralSuffix( n ),
( 1 == n ? "s" : "" ),
Util.DotOrColon( n ) );
string id_list = 0 == n
? string.Empty
: string.Join( ", ",
intersectingStructuralFramingElements
.Select(
x => x.Id.IntegerValue.ToString() ) )
+ ".";
Util.InfoMsg2( result, id_list );
return intersectingStructuralFramingElements;
}
```