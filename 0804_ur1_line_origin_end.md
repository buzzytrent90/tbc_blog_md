---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: qa
optimization_date: '2025-12-11T11:44:14.626139'
original_url: https://thebuildingcoder.typepad.com/blog/0804_ur1_line_origin_end.html
post_number: 0804
reading_time_minutes: 4
series: general
slug: ur1_line_origin_end
source_file: 0804_ur1_line_origin_end.htm
tags:
- csharp
- elements
- family
- geometry
- parameters
- references
- revit-api
- transactions
- views
- walls
title: UR1 and Line Origin versus Start and End Point
word_count: 865
---

### UR1 and Line Origin versus Start and End Point

My official vacation time has begun.
I still have rather a backlog of important, pending, almost completed projects, unanswered blog comments, and unavoidable meetings, so I will be online now and then in spite of that.

I also want to keep us all updated on important little titbits such as the UR1:

#### Revit 2013 Update Release 1

The Revit 2013 Update Release 1 is now available for download and installation from the web and within the product through Communication Center.
Here are links to the appropriate update pages:

- [Revit 2013 UR1](http://usa.autodesk.com/adsk/servlet/ps/dl/item?siteID=123112&id=20121968&linkID=16831210)- [Revit Architecture 2013 UR1](http://usa.autodesk.com/adsk/servlet/ps/dl/item?id=20121970&siteID=123112&linkID=9273944)- [Revit MEP 2013 UR1](http://usa.autodesk.com/adsk/servlet/ps/dl/item?id=20122070&siteID=123112&linkID=12828318)- [Revit Structure 2013 UR1](http://usa.autodesk.com/adsk/servlet/ps/dl/item?id=20122015&siteID=123112&linkID=9280927)

The build identifier is 20120716\_1115.

Here is a list of the API enhancements implemented in this update:

- Allows Document.PostFailure to be used to post multiple errors during a single transaction that do
  not reference an ElementId.- Improves stability using UIApplication.DoDragDrop when a Revit command (such as the Wall
    tool) was active.- Improves stability by disabling keyboard shortcuts (except view zoom shortcuts) when a
      PreviewControl is active.- Improves stability in ReferenceIntersector.FindNearest() when no matching target pick is found.- Corrects validation logic to allow NewFamilyInstance to place face-based families on transformed
          family instances.- Dimension.Above and Dimension.Below now update the dimension after their data is changed
            without requiring any user action.- The properties MechanicalSystem.SystemType, ElectricalConnector.SystemType,
              PipeConnector.SystemType are obsolete in Revit 2013. Instead query the parameter
              RBS\_DUCT\_CONNECTOR\_SYSTEM\_CLASSIFICATION\_PARAM on ConnectorElement.- Corrects data reported with ConnectorManager.UnusedConnectors.- Fixes a file corruption that could occur when extensible storage data was added to an element in
                  a central file.- Improves stability when saving a file with extensible storage data that overwrites an existing file
                    that also contains extensible storage.- RVT Links created with RevitLinkType.Create will remain loaded when the RVT containing the
                      link is reopened.- Previously, setting 'suppressBendRadius' to true in method Rebar.GetCenterlineCurves() would
                        cause both fillet bends and user-drawn, parameterized arcs to be omitted from the collection of
                        curves returned by the method. The method now omits only the fillet bends; the drawn arcs are
                        included along with the straight edges.- Updates Rebar.GetCenterlineCurves() method with an additional argument: a MultiplanarOption
                          (enum), which should be set to IncludeAllMultiplanarCurves or IncludeOnlyPlanarCurves. This
                          argument controls whether all curves of a multi-planar Rebar instance are returned, or only those
                          which lie in the primary plane.- Enables method Rebar.ComputeDrivingCurves(). This method returns a collection of curves that
                            includes the lines and arcs that drive the shape, but excludes fillets and hooks. It is equivalent to
                            calling GetCenterlinCurves(adjustForSelfIntersection=false, suppressHooks=true,
                            suppressBendRadius=true, multiplanarOption=IncludeOnlyPlanarCurves)- Improvements have been made in RebarShape methods that deal with matching RebarShapes to
                              collections of curves: CreateFromCurvesAndShape(), RebarShapeMatchesCurvesAndHooks().- Corrects behavior ot RebarShape.Create() method to not ignore the out-of-plane bend diameter
                                specified in the RebarShapeMultiplanarDefinition argument object, and always used an internal
                                default value.

Meanwhile, here is a neat little interesting and basic geometrical Revit API question that arose last week:

#### Relationship of Line Origin, Start and End Point

**Question:** When examining the AnalyticalSurface of a floor in my sample model, I noticed that the Origin property of a certain Line instance is not equal to either of its end points returned by the EndPoint property (get\_EndPoint method in C#).

Until now, I had assumed that the Origin property of a line is always equal to one of its end points.

Could you please clarify:

1. What exactly is the definition of a line's Origin property?- How does the line origin relate to its end points?

**Answer:** I already presented some basic facts on the Revit API
[lines, curves](http://thebuildingcoder.typepad.com/blog/2010/01/curves.html) and their
[parameterisations](http://thebuildingcoder.typepad.com/blog/2010/01/curve-parameterisation.html) from
Scott Conover's AU 2009 class on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html),
which he has continued updating, most recently for the
[AEC DevCamp 2012](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-one.html#4)
([finalised material](http://thebuildingcoder.typepad.com/blog/2012/06/obj-model-exporter-take-one.html#2)).

To answer your questions directly:

1. The line origin and direction define the location of the infinite unbounded line.
   The start and end point of the bounded line can lie anywhere along this line.- The origin always lies somewhere on the infinite unbounded line.
     There is no guarantee that it will coincide with either the start or end point of the bounded line, though.
     The origin of a bounded line may even lie outside the bounded part of the infinite line.

I hope this helps clarify things.

Anyway, now I'm off to the sunny
[Provence](http://en.wikipedia.org/wiki/Provence)!