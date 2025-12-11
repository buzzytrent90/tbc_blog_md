---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:16.311400'
original_url: https://thebuildingcoder.typepad.com/blog/1571_void_inst_cut_floor.html
post_number: '1571'
reading_time_minutes: 3
series: general
slug: void_inst_cut_floor
source_file: 1571_void_inst_cut_floor.md
tags:
- elements
- family
- filtering
- geometry
- levels
- parameters
- revit-api
- sheets
title: Void Inst Cut Floor
word_count: 539
---

### FindInserts Determines Void Instances Cutting Floor
Is it hot enough for you?
It sure is for this guy:
![Melted candle](img/221_melted_candle_400.jpg)
Time for some rest and recuperation, meseems...
Before that, let me share another brilliant and super succinct solution provided by Fair59, answering
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread on how
to [get cutting void instances in the floor](https://forums.autodesk.com/t5/revit-api-forum/get-cutting-void-instances-in-the-floor/m-p/7170237) using
the [`HostObject`](http://www.revitapidocs.com/2017/56a32e0b-df65-a6ba-40bd-8f50a1f31dcd.htm)
[`FindInserts`](http://www.revitapidocs.com/2017/58990230-38cb-3af7-fd25-96ed3215a43d.htm) method:
\*\*Question:\*\* I have a floor on which a family instance is inserted on the face of the floor (the instance host is also the floor).
I checked in the family the "Cut with Void When Loaded" parameter, so that the void is created in the floor.
Now, I want to retrieve all the instances that create voids in the floor.
I did some research, and found the discussion
of [Boolean operations and `InstanceVoidCutUtils`](http://thebuildingcoder.typepad.com/blog/2011/06/boolean-operations-and-instancevoidcututils.html).
But when I use the `InstanceVoidCutUtils` `GetCuttingVoidInstances` method, it returns an empty list.
I also looked
at the [`ElementIntersectsSolidFilter` problem and solution](http://thebuildingcoder.typepad.com/blog/2015/07/intersect-solid-filter-avf-and-directshape-for-debugging.html#2) and
tried `ElementIntersectsElementFilter` and `ElementIntersectsSolidFilter`.
Those filters do not return the expected result for me to deduce the voids in the floor either; in fact, they say that no elements intersect.
First case – area = 607.558m2 and Volume = 243.023m3:
![Void instances cutting floor](img/void_inst_cut_floor_1.png)
Second case – area = 607.558m2 and Volume = 243.023m3:
![Void instances cutting floor](img/void_inst_cut_floor_2.png)
`Family` parameter "Cut with Voids When Loaded":
![Void instances cutting floor](img/void_inst_cut_floor_3.png)
`FamilyInstance` cutting host:
![Void instances cutting floor](img/void_inst_cut_floor_4.png)
Here is the code I use:
```csharp
Solid solid = floor.get_Geometry( new Options() )
.OfType()
.Where( s => (null != s) && (!s.Edges.IsEmpty) )
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
floor ) );
int n = intersectingInstances.Count();
```
Here, both `n` and `n1` are equal to 0.
\*\*Answer:\*\* Try using
the [`HostObject`](http://www.revitapidocs.com/2017/56a32e0b-df65-a6ba-40bd-8f50a1f31dcd.htm)
[`FindInserts`](http://www.revitapidocs.com/2017/58990230-38cb-3af7-fd25-96ed3215a43d.htm) method instead:
```csharp
HostObject floor;
List intersectingInstanceIds
= floor.FindInserts( false, false, false, true )
.ToList();
```
\*\*Response:\*\* I have done some tests and here are my results:
![Void instances cutting floor](img/void_inst_cut_floor_6.png)
Situation:
- `Fl_1` is hosted by Level 3 and intersects the floor.
- `Fl_2` is hosted by the floor and intersects it.
Results:
1. Do not cut geometry:
- `InstanceVoidCutUtils.GetCuttingVoidInstances(floor)` returns `void`
- `floor.FindInserts(false,false,false,true)` returns `Fl_2`
2. Cut geometry:
- `InstanceVoidCutUtils.GetCuttingVoidInstances(floor)` returns `Fl_1`
- `floor.FindInserts(false,false,false,true)` returns both `Fl_1` and `Fl_2`
In summary, `FindInserts` returns `FI_1` even if its host (Level 3) is not the floor.
It's good.
I think we can say that the problem is solved.
Thank you FAIR59 ;)