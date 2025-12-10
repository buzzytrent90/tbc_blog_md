---
post_number: "0463"
title: "Filtered Element Collectors"
slug: "filtered_element_collector"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'revit-api', 'rooms', 'schedules', 'transactions', 'views', 'walls', 'windows']
source_file: "0463_filtered_element_collector.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0463_filtered_element_collector.html"
---

### Filtered Element Collectors

One of the most important enhancements made in the Revit 2011 API which is guaranteed to affect almost every single conceivable Revit add-in is the new
[filtered element collector](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) mechanism, and we have already looked at numerous aspects and examples of using it in various ways.

It represents the one and only way to access elements in the Revit database, if you disregard a number of dedicated Document properties providing direct access to specialised collections of various types, which can easily be replaced by filtered element collectors as well.

In addition to all the results published here so far, I was astounded by the wealth of yet more exciting and useful information and benchmarking results presented by Kevin Vandecar, ex Principal Engineer of the Revit API Team, at the
[AEC DevCamp](http://thebuildingcoder.typepad.com/blog/2010/03/devcamp-devlabs-and-updated-api-training-schedule.html) conference
in June 2010 in his presentation on 'Revit Performance Tips and Tricks – Exploiting 2011 API Features'.
Kevin has since rejoined the ADN DevTech team, where he specialises on the M & E APIs and is impressed by the powerful and fascinating programming possibilities they offer.
Here is his session summary:

In this class we will cover techniques that can be used to efficiently find elements and information from a Revit model. This new framework was introduced in the 2011 release and includes much more flexibility than before. Along with the flexibility comes some complexity, so we will shed light on the best approaches for common element gathering tasks. We will discuss the various new filters including logical and inverted aspects. We will also show how the iteration framework can be used with the .NET Framework LINQ APIs. We will discuss Regeneration topics including "when" regeneration should be done. Finally there are a few auto-join performance tips that we will share.

Here are Kevin's complete session materials
[AecDevCamp2010-RevitPerformanceTipsTricks.zip](zip/AecDevCamp2010-RevitPerformanceTipsTricks.zip),
which include the presentation, the hand-out document, a C# .NET sample add-in, and a couple of Revit test models.

Unfortunately, Kevin will not be able to present this material himself at Autodesk University.
It would be a great shame if it were not shown at all, so I have been asked to present it for him instead, which I am very much looking forward to doing.
It will be part of my class **CP234-2**.
Here are the complete descriptions of all three of
[my AU 2010 classes](http://thebuildingcoder.typepad.com/blog/2010/09/autodesk-university-2010-classes.html).

In the best spirit of Autodesk University, Kevin wrote a hand-out document covering the entire session content in some detail.
His handout is presented in full below.
In one of the upcoming posts, I would like to present some details of his benchmarking and example application together with an overview of the other existing samples and results published so far on the blog:

### Performance Tips and Tricks for the Revit 2011 API

#### Introduction

This course covers tips and techniques to efficiently filter and iterate elements within the Revit model. The system is designed to ensure that you determine what you really need to find before iterating so that there is an implicit performance gain over previous techniques. By combining multiple criteria, the new framework works efficiently internally to find a subset of the model that can then be iterated to perform common tasks. The course also covers regeneration and auto-join aspects of the API.

If you are not already an Autodesk Developer Network ADN member, see this
[information about ADN](http://www.autodesk.com/adn).

We also post periodically SDK updates and other Revit API information to the
[Revit Developer Center](http://www.autodesk.com/developrevit).

#### Sample Code

The sample code that accompanies this course is meant mainly to illustrate techniques and performance aspects using different techniques. Because of this, there is minimal error checking and exception handling. If reusing any of the sample code in your production code, please remember to add appropriate error handling.

The sample code also uses the .NET framework timing utility class called Stopwatch. This is part of the Microsoft .NET System.Diagnostics namespace and is an easy-to-use facility for timing operations. The Stopwatch class was built by MS using low-level API calls, with less overhead than other .NET methods. If the hardware and Windows version of the computer support a high-resolution performance counter, it will use this counter instead of the standard PC clock.

#### Revit 2011 API Filtering Framework

The 2011 release introduces a new filtering and iteration framework.
The internal implementation is done on native side so matching takes place before the managed wrapper is created.
This ensures the greatest performance when finding elements with certain criteria.
The built-in filters provide this criteria and then you can iterate or filter further using built-in iteration APIs, or with LINQ. In providing the built-in filters, it removes much of the per-element access that would marshal between native internals and the managed API.

This removes much of the per-element access that marshals between native internals and managed API.

#### Filtering Elements

The process of finding elements that match your search criteria is easier than ever with Revit 2011. The new filtering framework process can be broken down into three basic steps:

1. Filtering is the base step – first decide what you want
   - Find all elements with the model, a given set, or within a view-only- You can easily combine multiple criteria- Use shortcuts for common criteria- Combine with another collector through Intersection/Union- Add exclusions for things you know should not be included- Choose how to access
     - Choose a built-in iterator for Elements or ElementIds- Get and Element set or ElementId set- Optionally use the IEnumerable interface, for example with LINQ- Access the found elements
       - Use built-in languages features (foreach, while, etc.)- Use built-in facility to get first element that matches (Element or ElementId)- Optionally use LINQ

#### Example Results

To demonstrate the enhanced performance of 2011 over 2010, some internal testing provided performance results for some of the filters as follows, listing the execution time of various filters in milliseconds in 2010 and 2011 and the performance gain determined by the relation between the two:

| Example | 2010 ms | 2011 ms | Gain |
| --- | --- | --- | --- |
| Filter for each API type | 4828 | 618 | 7.8 |
| All elements of ElementType | 7 | 5 | 1.4 |
| Elements of Category | 2588 | 789 | 3.2 |
| Elements by Parameter filter | 49 | 20 | 2.4 |
| Elements of StructuralType | 86 | 18 | 4.7 |
| Elements of StructuralUsage | 210 | 43 | 4.8 |
| Elements of Structural MaterialTypes | 116 | 16 | 7.2 |
| Walls of Structural WallUsages | 70 | 13 | 5.4 |
| Enclosures and Tags | 112 | 20 | 5.6 |
| Curve-Driven Elements | 53 | 3 | 17.7 |
| Elements of Design Options | 51 | 35 | 1.5 |
| Elements on Levels | 74 | 21 | 3.5 |

#### FilteredElementCollector

At the heart of the system is the FilteredElementCollector class.
It provides the collection facility based on applied filters and then provides the interfaces to access the found elements.
The documentation is provided here for convenience.

Developers can assign a variety of conditions to filter the elements which are returned. This class requires that at least one condition be set before making the attempt to access the elements.

Revit will attempt to organize the filters in order to minimize expansion of elements regardless of the order in which conditions and filters are applied.

There are three groups of methods which you can use on a given collector once you have applied filter(s) to it. One group provides collections of all passing elements, a second finds the first match of the given filter(s), and a third provides an iterator which is evaluated lazily (each element is tested by the filter only when the iterator reaches it). You should only use one of the methods from these groups at a time; the collector will reset if you call another method to extract elements. Thus, if you have previously obtained an iterator, it will be stopped and traverse no more elements if you call another method to extract elements.

In .NET, this class supports the IEnumerable interface for Elements. You can use this class with LINQ queries and operations to process lists of elements. Note that because the ElementFilters and the shortcut methods offered by this class process elements in native code before their managed wrappers are generated, better performance will be obtained by using as many native filters as possible on the collector before attempting to process the results using LINQ queries.

One special consideration when using this class in .NET: the debugger will attempt to traverse the members of the collector because of its implementation of IEnumerable. You may see strange results if you also attempt to extract the first element or all elements from the collector while the debugger is also looking at the contents of the collector.

Tip: Do NOT use the debugger to inspect the element list during IEnumerable access (i.e. through LINQ).
As the statement above mentions, strange results can occur.
In fact during the creation of this material, it was observed a crash in Revit simply due to the debugger trying to inspect the contents of the list.

#### Filter Types

Filters are applied to the collector to define the search criteria.
The base class is ElementFilter, and three derived classes provide the different types of filters that can be applied.

- ElementLogicalFilter – Logical filters combine two or more filters logically with specific functionality provided by And and Or logical filters. Note that the component filters may be reordered by Revit to cause the quickest acting filters to be evaluated first.- ElementQuickFilter – Quick filters use an internal element record to determine passing state. This allows Revit to find elements which have not been expanded into internal memory yet.- ElementSlowFilter – Slow filters will expand the element into memory. Not all information can be obtained by the element record, so these filters must expand to determine passing state.

Tip: It is better to couple slow filters with at least one ElementQuickFilter (e.g. class filter), to minimize the number of elements expanded in order to evaluate against the criteria set by this filter.

#### Efficiency Guidelines

There is no special need to apply quick filters before slow ones, since Revit internally optimizes the order automatically, pushing the quick filters first, regardless of the external order specified. Because the actual iteration takes place only once based on the filter criteria, this optimization can be performed internally.

It is however important to use as many quick filters as possible to reduce the number of elements before any slow filters are applied. Some internal reordering may take place, but this still ensures best performance.

When you apply slow filters, the quick ones will already have eliminated a large number of elements, so the slow filters will not have such a performance impact.

After using the built-in filtering techniques to maximum capacity, you may want to consider using LINQ or explicit coding to narrow the results down further. Be aware that slow filters will normally still be faster than explicit iteration over the results and application of LINQ or custom queries.

Tip: Use the shortcut methods on FilteredElementCollector. Because there are currently no shortcuts for slow filters, you can be sure when using a shortcut you are getting a quick filter. Examples:

- OfClass- OfCategoryId

#### Logical Filters

There are two logical filters representing Boolean AND and OR statements.
Both can combine the results of two or more input filters:

- LogicalAndFilter: Elements must pass two or more filters; shortcuts: WherePasses adds one additional filter; IntersectWith joins two sets of independent filters.- LogicalOrFilter: Elements must pass at least one of two or more filters; shortcuts: UnionWith joins two sets of independent filters

#### Quick Filters

Here is a list of all quick filters with their passing criteria and shortcut methods, if available:

- ElementCategoryFilter: Elements matching the input category id; shortcut OfCategoryId- ElementClassFilter: Elements matching the input runtime class; shortcut OfClass- ElementIsElementTypeFilter: Elements which are "Element types" (symbols); shortcuts WhereElementIsElementType, WhereElementIsNotElementType- ElementOwnerViewFilter: Elements which are view-specific; shortcuts OwnedByView, WhereElementIsViewIndependent- ElementDesignOptionFilter: Elements in a particular design option; shortcut ContainedInDesignOption- ElementIsCurveDrivenFilter: Elements which are curve driven; shortcut WhereElementIsCurveDriven- ElementStructuralTypeFilter: Elements matching the given structural type ; no shortcut- FamilySymbolFilter: Symbols of a particular family; no shortcut- ExclusionFilter: All elements except the element ids input to the filter; shortcut Excluding- BoundingBoxIntersectsFilter: Elements which have a bounding box which intersects a given outline; no shortcut- BoundingBoxIsInsideFilter: Elements which have a bounding box inside a given outline; no shortcut- BoundingBoxContainsPointFilter: Elements which have a bounding box that contain a given point; no shortcut

#### Slow Filters

Here is a list of all slow filters with their passing criteria. As said, none of these have any shortcut methods defined:

- FamilyInstanceFilter: Instances of a particular family symbol- ElementLevelFilter: Elements associated to a given level id- ElementParameterFilter: Parameter existence, value matching, range matching, and/or string matching- PrimaryDesignOptionMemberFilter: Elements owned by any primary design option- StructuralInstanceUsageFilter: Structural usage parameter for FamilyInstances- StructuralWallUsageFilter: Structural usage parameter for Walls- StructuralMaterialTypeFilter: Material type applied to FamilyInstances- RoomFilter: Finds rooms- SpaceFilter: Finds spaces- AreaFilter: Finds areas- RoomTagFilter: Finds room tags- SpaceTagFilter: Finds space tags- AreaTagFilter: Finds area tags- CurveElementFilter: Finds specific types of curve elements (model curves, symbolic curves, detail curves, etc.)

#### Variances of the Filters – Tips/Tricks

Remember the filters have many capabilities. The follow functionality may not be obvious, but can be very powerful:

- Select the correct collector constructor (i.e. if you already have a set of elements to search, or only care about elements in a specific view)- Use filters in complex combinations, remembering to place slow filters last.- Use multiple filters to hone in on something.- Make use of inverted filters.

#### ElementParameterFilter

The ElementParameterFilter is a unique filter designed for use with parameters.
Benchmarking examples show that a parameter filter may be faster by a factor of two than post-processing results using explicit coding or LINQ to find a specific subset of elements.
Although it is a slow filter, it is advised to consider depending on the search criteria.

Tip: Use Revit Lookup to find the API name for the desired parameter.

#### Using LINQ

The FilteredElementCollector implements the IEnumerable interface which allows it to work with other .NET framework facilities that can use it. For example, the LINQ APIs are very powerful at allowing you to further find specific data. LINQ is short for the .NET framework Language-INtegrated Query, set, and transform operations.
For more information see
[this link](http://msdn.microsoft.com/en-us/netframework/aa904594.aspx).

LINQ can be used to further filter information using a query, or make use of its other features like XML or SQL.

#### Regeneration and Auto-Join

From the API Developer's Guide:

After new elements are created or elements are modified, regeneration and auto-joining of elements is required to propagate the changes throughout the model. Without a regeneration (and auto-join, when relevant), the Geometry property and the AnalyticalModel for Elements are either unobtainable (in the case of creating a new element) or they may be invalid. It is important to understand how and when regeneration occurs before accessing the Geometry or AnalyticalModel of an Element.

Although regeneration and auto-join are necessary to propagate changes made in the model, it can be time consuming. It is best if these events occur only as often as necessary. When using RegenerationOption.Automatic, regeneration and auto-joining happen automatically after each and every API call that modifies the model. In this case, the Geometry and AnalyticalModel are assigned immediately and no further action is required prior to accessing them. However, due to the slow performance of this option, it is obsolete and will be removed in a future version of Revit.

In RegenerationOption.Manual, regeneration and auto-joining occur automatically when a transaction that modifies the model is committed successfully, or whenever the Document.Regenerate or Document.AutoJoinElements methods are called. Regenerate and AutoJoinElements may only be called inside an open transaction. It should be noted that the Regeneration method can fail, in which case the RegenerationFailedException will be thrown. If this happens, the changes to the document need to be rolled back by rolling back the current transaction or subtransaction.

#### Regeneration

Regeneration access has been improved in 2011, but it means you need to be more aware of its consequences. In prior releases, regeneration was something that you had to control in, sort-of a 'reverse way'. For example there was a suspend mode, or a special batch create mode that were meant to help performance. With the 2011 release, you should now apply the manual attribute to your Revit API entry point functions and call Document.Regenerate when it is needed. The automatic mode will be removed in the future because it is very inefficient. From the 2011 documentation:

- RegenerationOption.Automatic – The API framework will regenerate after every model level change (equivalent behaviour with Revit 2010 and earlier). Regeneration and update can be suspended using SuspendUpdating for some operations, but in general the performance of multiple modifications within the same file will be slower than RegenerationOption.Manual. This mode is provided for behavioural equivalence with Revit 2010 and earlier; it is obsolete and will be removed in a future release.- RegenerationOption.Manual – The API framework will not regenerate after every model level change. Instead, you may use the regeneration APIs to force update of the document after a group of changes. SuspendUpdating blocks are unnecessary and should not be used. Performance of multiple modifications of the Revit document should be faster than RegenerationOption.Automatic. Because this mode suspends all updates to the document, your application should not read data from the document after it has been modified until the document has been regenerated, or it runs the risk of accessing stale data. This mode will be only option in a future release.

#### Manual Regeneration Option

For 2011 and the future you should get into the habit of using the manual option and Document.Regenerate.
Consider your performance options as you move over to it.
The main point is that you should use regenerate only when needed.
This will improve the performance of modification routines and allow you to group or queue modifications until necessary to regenerate. Remember that manual mode will never be slower than the old automatic option. The only problem is the risk of missing a regenerate call and retrieving stale data. Regenerating more often than necessary should not always be considered a problem and does not cause too much overhead. The regeneration algorithm is optimized to minimize the regeneration of only those items that need it. However, remember that it is another operation, and it is advisable to experiment with your particular algorithm and typical data sets to understand the best performance. The performance of regeneration will depend on the magnitude of change and the complexity of the model – there is no simple and generic answer about how long regeneration will take.

#### Tips and Tricks

- Again, regenerating more often than necessary should not always be considered a problem and does not cause too much overhead. The regeneration algorithm is optimized to minimize the regeneration of only those items that need it. Regeneration when nothing needs to be regenerated is rather fast, hence it is not so costly – still better than Automatic. If in doubt, do it (i.e. when seeing errors pertaining to data access, this is the first thing to try in your code).- It is another operation, though! It is advisable to experiment with your particular algorithm and typical data sets to understand the best performance.- Querying data does not require regenerations unless there was a previous change to the part of the model that could affect the data being queried – that is sometimes hard to estimate, though.

#### Auto-Join

As mentioned above, auto-join is something that needs to be done when creating or modifying elements to ensure the geometry aspects are updated appropriately before accessing those elements. The trick is to know when to call it. Generally, you must call it when elements are related by geometry to other elements. For example, when two walls are added and a corner is formed, those elements are related at the corner and need auto-join.

Tip: With RegenerationOption.Automatic, elements that are meant to be auto-joined should automatically be updated.
There was a problem with this in the very first version of 2011 that was fixed in the first user release update.

#### Next Instalment

As said, one of the next topics will be an overview of Kevin's sample code and collector benchmarks, followed by a global overview of all the filtering samples we have studied so far.