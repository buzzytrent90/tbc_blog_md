---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: qa
optimization_date: '2025-12-11T11:44:16.782847'
original_url: https://thebuildingcoder.typepad.com/blog/1806_effective_filter.html
post_number: '1806'
reading_time_minutes: 7
series: filtering
slug: effective_filter
source_file: 1806_effective_filter.md
tags:
- elements
- filtering
- geometry
- parameters
- revit-api
- rooms
- sheets
- views
title: Effective Filter
word_count: 1393
---

### Forge Rooms, Effective Filtered Element Collectors
I successfully made it from Switzerland to Paris by train yesterday, in spite of the strikes here.
Now I am happily occupied with the Forge accelerator in the Autodesk office.
My only worry is how to get back again tomorrow.
This time again, the train I had originally booked has been cancelled.
We'll see how it goes.
Meanwhile, let's take a look at:
- [DA4R room support and new samples](#2)
- [Effective filtered element collection](#3)
- [Don't waste time optimising prematurely](#4)
#### DA4R Room Support and New Samples
Some recent [Forge Design Automation for Revit or DA4R](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.55) news
and samples:
Most importantly,
a [new RVT to SVF model derivative parameter generates additional content, including rooms and spaces](https://forge.autodesk.com/blog/new-rvt-svf-model-derivative-parameter-generates-additional-content-including-rooms-and-spaces).
Lots of other exciting [new samples](https://forge.autodesk.com/categories/code-samples) are available, including:
- Run additional programs inside a WorkItem
- Get number of BIM 360 projects per user
- Working with 2D and 3D scenes and geometry in Forge Viewer
- Fast track your cross-platform React native Forge app with the Expo SDK
- How to convert your plugin to work with Design Automation API for AutoCAD
- Go semantic with Viewer-custom web component for Forge Viewer
- New RVT->SVF Model Derivative parameter generates additional content, including rooms and spaces
- Combining PointClouds and Revit models in Forge Viewer
- Switch to orthographic corner view
- Custom BIM 360 search with Elasticsearch
- Get volume and surface area in the Viewer
- Run command from an AutoCAD AppBundle
- Run iLogic Rule without AppBundle
Here is a [link to view all samples](https://forge.autodesk.com/categories/code-samples).
#### Effective Filtered Element Collection
Back to the desktop Revit API, one issue that almost every Revit add-in developer faces is the efficiency of filtered element collectors.
Several interesting aspects are pointed out (and repeated) in the thread on
an [efficient way to check if an element exists in a view](https://forums.autodesk.com/t5/revit-api-forum/efficient-way-to-check-if-an-element-exists-in-a-view/m-p/9187613):
\*\*Question:\*\* I'm creating a list of views that contain .dwg ImportInstance(s).
For each view in the document, I'm using a `FilteredElementCollector` to get a list of elements meeting the criteria; if this list is not empty, the view is added to the list:
```csharp
foreach( Element e in viewElements )
{
View view = (View) e;
var stopwatch = new Stopwatch();
stopwatch.Start();
List elementsInView
= new FilteredElementCollector( doc, view.Id )
.OfClass( typeof( ImportInstance ) )
.Where( e => e.Category.Name.EndsWith( ".dwg" ) )
.OfType()
.ToList();
stopwatch.Stop();
Debug.WriteLine( view.Name + ": "
+ stopwatch.ElapsedMilliseconds + "ms" );
// if the current view contains at least 1 DWG
// ImportInstance, add the view to the list
if( elementsInView.Count > 0 )
{
viewsWithCAD.Add( view );
continue;
}
}
```
The FilteredElementCollector can understandably take more than 4000 ms to collect elements from a view containing many elements.
My goal is only to see if a single element exists in a view – not to collect all of the elements meeting the criteria; if I could make the FilteredElementCollector stop immediately after finding an element meeting the criteria, that would be helpful.
I would appreciate any advice on how to achieve this more efficiently.
Thank you.
\*\*Answer by Fair59\*\*, [Frank Aarssen](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518):
Stopping the collector at the first element:
```csharp
Element e1
= new FilteredElementCollector( doc, view.Id )
.OfClass( typeof( ImportInstance ) )
.FirstElement();
```
Possible further speed improvement:
- Select all `ImportInstance` elements
- If it is not a DWG: exclude from collector in view-loop → no need for `category.Name` check
- Else: if DWG is ViewSpecific, i.e., is "2D annotation" in view → `ownerview` contains DWG, can be added to `viewsWithCAD`, and can be excluded from view-loop
```csharp
IEnumerable instances
= new FilteredElementCollector( doc )
.OfClass( typeof( ImportInstance ) )
.Cast();
List toExclude = new List();
foreach( ImportInstance instance in instances )
{
if( !instance.Category.Name.EndsWith( ".dwg" ) )
{
toExclude.Add( instance.Id );
continue;
}
if( instance.ViewSpecific ) // dwg only exists in ownerview
{
View ownerview = doc.GetElement( instance.OwnerViewId ) as View;
viewsWithCAD.Add( ownerview );
if( viewElements.Contains( ownerview ) )
viewElements.Remove( ownerview );
}
}
foreach( Element e in viewElements )
{
View view = (View) e;
var stopwatch = new Stopwatch();
stopwatch.Start();
Element e1 = null;
if( toExclude.Count > 0 )
{
e1 = new FilteredElementCollector( doc, view.Id )
.Excluding( toExclude )
.OfClass( typeof( ImportInstance ) )
.FirstElement();
}
else
{
e1 = new FilteredElementCollector( doc, view.Id )
.OfClass( typeof( ImportInstance ) )
.FirstElement();
}
stopwatch.Stop();
Debug.WriteLine( view.Name + ": "
+ stopwatch.ElapsedMilliseconds + "ms" );
// if the current view contains at least 1 DWG
// ImportInstance, add the view to the list
if( e1 != null )
{
viewsWithCAD.Add( view );
}
}
```
Many thanks for the interesting question, and many thanks to Fair59 for yet another extremely knowledgeable and helpful solution!
\*\*Notes:\*\* I do keep pointing out that converting a filtered element collector to a List is an inefficient thing to do, if you can avoid it.
It forces the collector to retrieve all the data, convert it to the .NET memory space, duplicate it, costing time and space.
For the same reasons, it is much more efficient to test and apply as many filters as possible within the Revit memory space before passing any data across to .NET.
In this case, you can test the parameter values using a parameter filter instead of the LINQ post-processing that you are applying in you sample code snippet.
As Fair59 points out and we have discussed in the past, you can cancel a collector as soon as your target has been reached:
- [Aborting filtered element collection](https://thebuildingcoder.typepad.com/blog/2019/02/cancelling-filtered-element-collection.html)
So, you can save time and space in several ways:
- Use a parameter filter instead of LINQ or .NET post-processing
- Do not convert to a `List`
Both of these force the filtered element collector to retrieve and return all results.
Here is an explanation of the various types of filters versus post-processing in .NET:
- [Slow, slower still and faster filtering](https://thebuildingcoder.typepad.com/blog/2019/04/slow-slower-still-and-faster-filtering.html)
Here are some discussions and a benchmark of the results of using a parameter filter versus LINQ and .NET post-processing:
- [Filtering for a specific parameter value](https://thebuildingcoder.typepad.com/blog/2018/06/forge-tutorials-and-filtering-for-a-parameter-value.html#3)
- [Filtered element collector benchmark](https://thebuildingcoder.typepad.com/blog/2019/05/filtered-element-collector-benchmark.html#3)
We also discussed the issue of finding all views displaying an element a couple of times in the past:
- [Views displaying given element](https://thebuildingcoder.typepad.com/blog/2014/05/views-displaying-given-element-svg-and-nosql.html#6)
- [Determining views showing an element](https://thebuildingcoder.typepad.com/blog/2016/12/determining-views-showing-an-element.html)
- [Retrieving elements visible in view](https://thebuildingcoder.typepad.com/blog/2017/05/retrieving-elements-visible-in-view.html)
- [Can you avoid generating graphics?](https://thebuildingcoder.typepad.com/blog/2019/10/generating-graphics-and-collecting-assets.html#2)
![Harvester](img/harvester.png)
#### Don't Waste Time Optimising Prematurely
[Mastjaso](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1058186) adds
some important professional advice in
his [comment below](https://thebuildingcoder.typepad.com/blog/2019/12/forge-rooms-effective-filtered-element-collectors.html#comment-4719321388):
> Interesting discussion on filtered element collectors, though I have to say that I have personally spend far too much time worrying about the efficiencies of those collectors when it has turned out to be inconsequential.
> For new developers starting out, or ones tackling a new project for the first time, I'd remember the adage that premature optimization is the death of software.
I start basically all my projects with super basic `OfClass` filtered element collectors and then cast and convert them to the .NET memory space so that I can use LINQ which produces shorter, more readable, and easier to debug code than the filtered element collection API.
It's only near the end of a project, once features are settled that I'll start swapping out LINQ for more nuanced uses of the FEC, but even then I'll only do it \*if\* I need the performance boost somewhere.
Before optimising anything at all, benchmark or profile to find out where and whether there are any serious performance issues at all.