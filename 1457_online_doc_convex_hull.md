---
post_number: "1457"
title: "The Building Coder"
slug: "online_doc_convex_hull"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'levels', 'revit-api', 'rooms', 'schedules', 'sheets', 'views', 'walls']
source_file: "1457_online_doc_convex_hull.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1457_online_doc_convex_hull.html"
---

### Online Revit API Docs and Convex Hull
Today I happily present a brilliant piece of Revit API news on the documentation side of things, and another handy utility method for your Revit API programming toolbox:
- [Online Revit API documentation](#2)
- [2D convex hull algorithm in C# using `XYZ` points](#3)
#### Online Revit API Documentation
The contents of the Revit API help file RevitAPI.chm are finally available online.
And not only that, but the web site includes all three versions for Revit 2015, Revit 2016 and Revit 2017.
As you know, the two main pieces of Revit API documentation are the Revit API help file RevitAPI.chm, included with the Revit SDK, available from
the [Revit Developer Centre](http://www.autodesk.com/developrevit), and
the [developer guide](http://help.autodesk.com/view/RVT/2017/ENU/?guid=GUID-F0A122E0-E556-4D0D-9D0F-7E72A9315A42),
provided in the 'Developers' section of the
online [Revit Help](http://help.autodesk.com/view/RVT/2017/ENU).
The help file was not available online, though, which also means that it was not included in standard Internet searches
using [Google](https://www.google.com)
or [DuckDuckGo](https://duckduckgo.com).
With the notable exception
of [Revit 2014](http://thebuildingcoder.typepad.com/blog/2014/01/revit-api-help-online-and-hiking-on-la-palma.html),
[revitapisearch.com](http://revitapisearch.com),
implemented by Peter Boyer of the Dynamo team.
Just a few weeks ago, Arif Hanif expressed interest to do the same for Revit 2017 in
a [couple](http://thebuildingcoder.typepad.com/blog/2016/04/whats-new-in-the-revit-2017-api.html#comment-2823075296)
of [comments](http://thebuildingcoder.typepad.com/blog/2016/04/whats-new-in-the-revit-2017-api.html#comment-2824265506)
on [What's New in the Revit 2017 API](http://thebuildingcoder.typepad.com/blog/2016/04/whats-new-in-the-revit-2017-api.html).
Well, someone beat him to it.
In
his [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
on [Revit API Documentation Online ](http://forums.autodesk.com/t5/revit-api/revit-api-documentation-online/m-p/6495377),
[@gtalarico](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/67891) says:
> Since Google doesn't seem too excited to index my 60k+ page website, I wanted to share it here so people can find it and hopefully use it! : )
> It currently includes the full official documentation (from the CHM file) for APIS 2015, 2016, and 2017:

[www.revitapidocs.com](http://www.revitapidocs.com)
> It was a lot of planning and work, but came together faster and better than I expected.
![www.revitapidocs.com](img/revitapidocs.png)
Please let him know if you have any feedback on it.
Ever so many thanks to gtalarico for all his work and making this useful resource available to the global Revit API community!
#### Contributing and Implementation Details
Gtalarico added some additional info on the project:
The code is on github at [github.com/gtalarico/revitapidocs](https://github.com/gtalarico/revitapidocs).
The project is definately open to collaborators. Welcome!
It needs +code docs, +test coverage, and can probably be improved and optimized significantly by more seasoned web developers.
Regarding github pages, it could probably be done, but I haven't used it myself, so I don't know the limitations.
Here are some of the challenges and constraints:
1. Namespace Menu:
- Each API/year has an index with around 20K nested entries, sometimes many levels deep.
Performance can get tricky, and so is creating a good and responsive UI for browsing it, which is why I wanted it to be collapsible.
If I recall correctly, Readthedocs for instance, limits the depth of the menu.
2. Content:
- The content I had access to (.html files extracted from chm) were not pretty, so I had to do some unusual CSS overrides and eventually batch processed the 60k+ html files to remove unnecessary JS and html code to make the pages look good and perform well. I was also was concerned about appearance to google crawler (cleaned code, and added schema.org structured data on every page).
3. Performance:
- The namespace html file alone was almost 3MB and 140K lines of html code, which is not good.
To optimize it, I am serving the menu asynchronously as json, so it loads while the rest of the content is being built, and can be cached.
4. Built-in search:
- I originally tried using google custom search, but google can take a long time to index it (if it happens at all - 60k+ pages)
Even with a full sitemap, it will probably just take time, but I didn't want to wait.
So I replaced the Google Custom Search box with my own custom search.
I tried a JS client side search, similar to what git pages has, but it was crashing the browser (remember namespace is +100K lines), so I ended up pushing it server side which makes it reasonably fast, e.g., [www.revitapidocs.com/2015/search?query=viewschedule](http://www.revitapidocs.com/2015/search?query=viewschedule).
#### 2D Convex Hull Algorithm in C# using `XYZ`
Yesterday, I mentioned the convex hull calculation as one option for determining
the [bounding box of selected elements or entire model](http://thebuildingcoder.typepad.com/blog/2016/08/vacation-end-forge-news-and-bounding-boxes.html#8).
Maxence replied in
a [comment](http://thebuildingcoder.typepad.com/blog/2016/08/vacation-end-forge-news-and-bounding-boxes.html#comment-2839904399) on
that post and provided a convex hull implementation in C#.
It is a 2D algorithm implementing the [Jarvis march or Gift wrapping algorithm](https://en.wikipedia.org/wiki/Gift_wrapping_algorithm):
It makes use of an extension method `MinBy` on the generic `IEnumerable` class,
from [MoreLINQ](https://github.com/morelinq/MoreLINQ/blob/master/MoreLinq/MinBy.cs) by
Jonathan Skeet:
```csharp
public static class IEnumerableExtensions
{
public static tsource MinBy(
this IEnumerable source,
Func selector )
{
return source.MinBy( selector, Comparer.Default );
}
public static tsource MinBy(
this IEnumerable source,
Func selector,
IComparer comparer )
{
if( source == null ) throw new ArgumentNullException( nameof( source ) );
if( selector == null ) throw new ArgumentNullException( nameof( selector ) );
if( comparer == null ) throw new ArgumentNullException( nameof( comparer ) );
using( IEnumerator sourceIterator = source.GetEnumerator() )
{
if( !sourceIterator.MoveNext() )
throw new InvalidOperationException( "Sequence was empty" );
tsource min = sourceIterator.Current;
tkey minKey = selector( min );
while( sourceIterator.MoveNext() )
{
tsource candidate = sourceIterator.Current;
tkey candidateProjected = selector( candidate );
if( comparer.Compare( candidateProjected, minKey ) < 0 )
{
min = candidate;
minKey = candidateProjected;
}
}
return min;
}
}
}
```
With that helper method in hand, the convex hull implementation is quite short and sweet:
```csharp
#region Convex Hull
///
/// Return the convex hull of a list of points
/// using the Jarvis march or Gift wrapping:
/// https://en.wikipedia.org/wiki/Gift_wrapping_algorithm
/// Written by Maxence.
/// summary>
public static List ConvexHull( List points )
{
if( points == null ) throw new ArgumentNullException( nameof( points ) );
XYZ startPoint = points.MinBy( p => p.X );
var convexHullPoints = new List();
XYZ walkingPoint = startPoint;
XYZ refVector = XYZ.BasisY.Negate();
do
{
convexHullPoints.Add( walkingPoint );
XYZ wp = walkingPoint;
XYZ rv = refVector;
walkingPoint = points.MinBy( p =>
{
double angle = ( p - wp ).AngleOnPlaneTo( rv, XYZ.BasisZ );
if( angle < 1e-10 ) angle = 2 \* Math.PI;
return angle;
} );
refVector = wp - walkingPoint;
} while( walkingPoint != startPoint );
convexHullPoints.Reverse();
return convexHullPoints;
}
#endregion // Convex Hull
```
For testing purposes, I make use of it like this in the `CmdListAllRooms` external command:
```csharp
///
/// Return bounding box calculated from the room
/// boundary segments. The lower left corner turns
/// out to be identical with the one returned by
/// the standard room bounding box.
/// summary>
static List GetConvexHullOfRoomBoundary(
IList> boundary )
{
List pts = new List();
foreach( IList loop in boundary )
{
foreach( BoundarySegment seg in loop )
{
Curve c = seg.GetCurve();
pts.AddRange( c.Tessellate() );
}
}
int n = pts.Count;
pts = new List(
pts.Distinct( new CmdWallTopFaces
.XyzEqualityComparer( 1.0e-4 ) ) );
Debug.Print(
"{0} points from tessellated room boundaries, "
+ "{1} points after cleaning up duplicates",
n, pts.Count );
return Util.ConvexHull( pts);
}
```
Initially, I did not include the call to `Distinct`, which eliminates duplicate points returned by Revit that are intended to represent the same room corner but have small offsets from each other due to limited precision, or too high precision, whichever way you look at it.
Here are the
diffs [with just the pure convex hull implementation](https://github.com/jeremytammik/the_building_coder_samples/compare/2017.0.127.8...2017.0.127.9)
and [after adding the call to `Distinct`](https://github.com/jeremytammik/the_building_coder_samples/compare/2017.0.127.9...2017.0.127.10).
I tested it on the same ten rectangular rooms as yesterday and verified that their convex hulls correspond to their bounding boxes returned by Revit.
I also tested on the following squiggly room with some spline-shaped edges:
![Squiggly room](img/room_squiggly.png)
That returns the following results:

```
355 points from tessellated room boundaries, 324 points after cleaning up duplicates

Room nr. '1' named 'Room 1' at (-77.61,15.1,0) with lower left corner (-106.43,-8.65,0), convex hull (-104.93,-8.65,0), (-40.75,-8.51,0), (-40.41,33.07,0), (-106.43,33.27,0), bounding box ((-106.43,-8.65,0),(-40.41,33.27,13.12)) and area 1483.20391607451 sqf has 1 loop and 31 segments in first loop.
```

Everything discussed above is published
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2017.0.127.10](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.127.10).
Many thanks to Maxence for providing the nice convex hull algorithm implementation!