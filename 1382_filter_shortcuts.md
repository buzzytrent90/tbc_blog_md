---
post_number: "1382"
title: "Filter Shortcuts"
slug: "filter_shortcuts"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'python', 'revit-api', 'rooms', 'sheets', 'views', 'walls']
source_file: "1382_filter_shortcuts.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1382_filter_shortcuts.html"
---

### Quick, Slow and LINQ Element Filtering
I arrived back safe and sound in Switzerland after the exciting week at Autodesk University.
Another happy arrival is my first grandson:
![Grandson](/p/people/marlon/20151208_marlon_cropped.jpg)
He was preceded by his big sister almost two years ago.
Life goes on.
I have been wanting to highlight the difference between quick and slow element filters for quite a while, and now a query came in that provides an ideal opportunity.
- [Use of LINQ with filtered element collectors](#2)
- [Revit element filter classification](#3)
- [Filter types](#4)
- [Efficiency guidelines](#5)
- [Logical filters](#5)
- [Quick filters](#6)
- [Slow filters](#7)
#### Use of LINQ with Filtered Element Collectors
\*\*Question:\*\* I have been using lots of LINQ with delegates to perform database queries.
Now I am wondering how to use them with Revit filtered element collectors.
I see several different possibilities, e.g. like this:
![Filtered element collector sample code snippets](img/jo_collector_samples.png)
I have the following questions regarding this:
1. What is the primary difference/benefit between the sample 1 code, which uses the Revit `ElementFilter` objects, and the sample 3 code, which just uses pure LINQ on the FilteredElementCollector itself? I assume they would get the same results but I don't quite grasp when I should be using the native Revit filter objects versus pure LINQ queries.
2. In sample 2, is the `ToElements` method required? Is there any difference between that and the `foreach` loop on the FilteredElementCollector directly in sample 1?
3. Similar question 2, in sample 3 is it safe to perform LINQ on the FilteredElementCollector directly or should I be invoking `ToElements` first then performing LINQ?
I've found LINQ to be incredibly fast but I'm just trying to make sure I fully understand the impacts of using it on the FilteredElementCollector objects.
\*\*Answer:\*\* These are very relevant and important questions, I think.
However, I would \*always\* much prefer having the sample code snippets in text form rather than as a screen snapshot!
To start from the essentials and basics, you need to understand and consider three different levels of filtering element collector data access and performance:
- Quick filters
- Slow filters
- .NET filtering, aka post-processing
Quick filters are fast, because they operate on data present in the Revit element header.
In a huge project, Revit will not load the entire Revit database element information, only a small header for each element.
A quick filter can execute immediately using only the header information, which is always available in the project.
A slow filter may need to load the complete element information in order to execute. In a huge project, that may affect performance.
These two kinds of filters are built into the Revit filtered element collector framework and execute directly on the Revit element data within the Revit memory, with no need to translate and marshal the element information to pass it out to the .NET add-in client.
As soon as you use any kind of .NET operation to filter that data, you are requesting Revit to translate and marshal the element information and pass it out from the Revit memory space to the .NET add-in client. That is a hugely inefficient process, compared to the internal filtering.
Regardless of what you are searching for, doing it in .NET instead of using a built-in filter is guaranteed to double the execution time and halve the performance.
I tested various combinations of this quite extensively, starting with
the [collector benchmark](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) in 2010.
You can find more examples by searching the Internet for CmdCollectorPerformance, optionally adding 'revit api' or 'building coder'.
They refer to
the [CmdCollectorPerformance.cs module](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs)
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples).
Your question 2 regarding the optional use of `ToElements` is answered in my discussion of [ToElementIds performance](http://thebuildingcoder.typepad.com/blog/2012/12/toelementids-performance.html).
In short, you should avoid it unless you really need it.
One situation in which it might be useful is when you really need to determine the number of elements returned by the collector before starting to iterate over them. Unlike the collector itself, ToElements returns a generic List of Element objects providing a Count property.
A more common need might be to access the element ids.
I recently discussed an example that demonstrates the imperative need
to [close the collector before deleting elements](http://thebuildingcoder.typepad.com/blog/2015/12/au-ioc-banks-and-not-to-delete-while-iterating.html#2),
in which case `ToElementIds` can come in handy.
By now, I hope that it is clear that your sample 3 is an utter abomination and could be regarded as verging on criminal inefficiency.
If you say that you found LINQ to be incredibly fast, you will be happy to hear that the Revit filtered element collectors are at least twice as fast, in the worst case.
For the sake of completeness, see [below](#3) for more on the topic of the different Revit API filter types and quick versus slow ones.
An interesting specialised topic is how to convert from .NET post-process filtering to built-in Revit filtering in order to optimise performance. One typical situation in which the .NET filtering may seem easier to implement is when checking for specific element parameter values.
The collector performance samples pointed to above provide several samples of avoiding that by setting up appropriate element parameter filters instead, e.g., to [filter for elements in a specific view having a specific phase](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs#L148-L200).
Please also explore and understand the related discussion
on [FindElement and collector
optimisation](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html).
Now back to the overview that I have been planning to publish for so long:
#### Revit Element Filter Classification
Here is an overview of the different types of element filters, based on
the [Revit API Introduction slide deck](https://github.com/ADN-DevTech/RevitTrainingMaterial/blob/master/Presentation/1_Revit_API_Intro.pptx) from
the [ADN Revit API training material](https://github.com/ADN-DevTech/RevitTrainingMaterial).
All Revit database elements are bundled in one single container.
To retrieve an element of interest, you filter for it using a filtered element collector.
The `FilteredElementCollector` class is used to search, filter and iterate through a set of elements.
You can assign a variety of conditions to filter the elements that are returned.
The system requires that at least one condition be set before making the attempt to access the elements, otherwise an exception thrown.
The `FilteredElementCollector` class supports the `IEnumerable` interface, so you can iterate over the resulting elements directly using `foreach`.
\*\*Tip:\*\* because all built-in Revit element filters process elements in native code before their managed wrappers are generated, better performance will be obtained by using as many native filters as possible on the collector before attempting to process the results using LINQ queries or other .NET methods.
#### Filter Types
- Logical Filters – help to combine filter logic
- And
- Or
- Quick filters – use internal element record header to determine passing state
- Examples: ElementClassFilter, ElementCategoryFilter
- Slow filters – expand element to determine passing state
- Examples: FamilyInstanceFilter, AreaFilter
Quick filters can use the internal element record to determine passing state. This allows Revit to find elements that have not been expanded into internal memory yet.
Slow filters cannot determine the passing state based on the minimal element record, so these filters must load and expand each element to determine its passing state.
#### Efficiency Guidelines
- Filter quick aspects first
- Filter slow aspects later
- Do not use .NET or LINQ until after exhausting the built-in filtering techniques
\*\*Tip:\*\* Use the shortcut methods on FilteredElementCollector.
- Because there are currently no shortcuts for slow filters, you can be sure you are getting a quick filter when using a shortcut.
- Examples: OfClass, OfCategoryId
#### Logical Filters
- LogicalAndFilter: elements must pass two or more filters
- WherePasses – adds an additional filter
- IntersectWith – joins two sets of independent filters
- LogicalOrFilter – elements must pass at least one of two or more filters
- UnionWith – joins two sets of independent filters
#### Quick Filters
All quick filters are visible as shortcut methods on the filtered element collector.
Here is a list of the filter class name, shortcut method, if available, and description:
- ElementCategoryFilter – `OfCategoryId` – elements matching the input category id.
- ElementClassFilter – `OfClass` – elements matching the input runtime class.
- ElementIsElementTypeFilter – `WhereElementIsElementType`, `WhereElementIsNotElementType` – elements that are element types, also called family symbols.
- ElementOwnerViewFilter – `OwnedByView`, `WhereElementIsViewIndependent` – elements that are view-specific.
- ElementDesignOptionFilter – `ContainedInDesignOption` – elements in a particular design option.
- ElementIsCurveDrivenFilter – `WhereElementIsCurveDriven` – elements that are curve driven.
- ElementStructuralTypeFilter – none – elements matching the given structural type.
- FamilySymbolFilter – none – symbols of a particular family.
- ExclusionFilter – `Excluding` – all elements except the ones who's ids are passed into to the filter.
- BoundingBoxIntersectsFilter – none – elements that have a bounding box intersecting a given outline.
- BoundingBoxIsInsideFilter – none – elements that have a bounding box inside a given outline.
- BoundingBoxContainsPointFilter – none – elements that have a bounding box containing a given point.
#### Slow Filters
Slow filters have no corresponding shortcut method on the collector class:
- FamilyInstanceFilter – instances of a particular family symbol.
- ElementLevelFilter – elements associated to a given level id.
- ElementParameterFilter – parameter existence, value matching, range matching, and/or string matching.
- PrimaryDesignOptionMemberFilter – elements owned by any primary design option..
- StructuralInstanceUsageFilter – structural usage parameter for family instances.
- StructuralWallUsageFilter – structural usage parameter for walls.
- StructuralMaterialTypeFilter – material type applied to family instances.
- RoomFilter – finds rooms.
- SpaceFilter – finds spaces.
- AreaFilter – finds areas.
- RoomTagFilter – finds room tags.
- SpaceTagFilter – finds space tags.
- AreaTagFilter – finds area tags.
- CurveElementFilter – finds specific types of curve elements, e.g. model curves, symbolic curves, detail curves, etc.