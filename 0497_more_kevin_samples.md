---
post_number: "0497"
title: "More Kevin Filtering Benchmarks"
slug: "more_kevin_samples"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'levels', 'parameters', 'references', 'revit-api', 'schedules', 'transactions', 'views', 'walls']
source_file: "0497_more_kevin_samples.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0497_more_kevin_samples.html"
---

### More Kevin Filtering Benchmarks

I caught a germ yesterday in Moscow, maybe travelling on the subway to the office, or at the developer conference itself.
I suffered through the night with a pain in the bones of my face and a headache, and yet I was able to catch up on lots of sleep in the end, getting up much later than normal today.
The whole morning has also been very unpleasant inside my head, but the rest of me feels fine, and it seems to be getting better, miraculously.

The Russian developer conference yesterday was very active and lively with a lot of participation from the attendees.
I really enjoyed it very much.

Now I am sitting in the Domodedovo airport waiting for the flight to Paris via Vienna.
No rest for the wicked, as the English say.

In the overview of Kevin Vandecar's
[Revit performance tips and tricks](http://thebuildingcoder.typepad.com/blog/2010/10/filtered-element-collectors.html) presentation at the
[AEC DevCamp](http://thebuildingcoder.typepad.com/blog/2010/03/devcamp-devlabs-and-updated-api-training-schedule.html) conference
in Boston in June, which formed the base of my
[AU class CP234-2](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-university-2010-class-materials.html) on
the same topic, I promised to publish a list of the filtered element collector benchmarking samples he created.

I fulfilled part of that promise by discussing the
[level filter benchmark](http://thebuildingcoder.typepad.com/blog/2010/10/level-filter-benchmark.html) a
while ago, and the
[XML family usage report](http://thebuildingcoder.typepad.com/blog/2010/12/xml-family-usage-report.html) and
[intersecting element retrieval](http://thebuildingcoder.typepad.com/blog/2010/12/find-intersecting-elements.html) in
the past few days.
I added links to those posts in the list below.

Here is an overview of the rest of Kevin's example and benchmarking commands, which cover the following topics:

- Comparison of Revit 2011 filtered element collector versus pre-2011 element access- Basic filtering examples
    1. Structural material type and instance usage, Boolean combinations and inversion- [Bounding box intersection, view id and exclusion](http://thebuildingcoder.typepad.com/blog/2010/12/find-intersecting-elements.html)- LINQ examples
      1. Family instance, furniture category, material black textile- Non-native mullion class- [XML report of family and instance usage](http://thebuildingcoder.typepad.com/blog/2010/12/xml-family-usage-report.html)- Parameter filter samples
        1. Numeric evaluator and integer rule for specific named analysis display style- Ditto to test Boolean parameter for levels whose crop view is false- Numeric evaluator and double rule to find levels whose top offset is greater than Z- String evaluator and string rule to find elements whose view name contains "Level"- Parameter filter benchmark
          1. [Compare performance of level filter versus parameter filter versus LINQ](http://thebuildingcoder.typepad.com/blog/2010/10/level-filter-benchmark.html)- Regeneration and Auto-Join Examples
            1. Create walls without explicit auto-join call- Create walls with explicit auto-join call- Ditto with explicit regenerate and auto-join call for each element- Demonstrate stale data retrieval

Here is an updated archive
[AU10\_CP234-2\_Revit\_2011\_API\_Optimization.zip](/a/doc/revit/au/2010/doc/AU10_CP234-2_Revit_2011_API_Optimization.zip) of
my AU class material including the source code and Visual Studio solution.
I removed the sample models, since they are unchanged from my previous posting together with the class handout and presentation in my
[AU 2010 class materials](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-university-2010-class-materials.html).

As always, the sample code performs minimal error checking, and you are obviously encouraged to add adequate checks in your own code.

The commands are intended to be run in the three sample models provided: ArchSample.rvt, EmptyProject.rvt and StructuralUsage.rvt as indicated in the source code command descriptions.

The benchmarks make use of a simple timing utility class, the System.Diagnostics.Stopwatch class. It is built using low-level API calls and uses the high-resolution performance counter instead of standard PC clock, assuming hardware and OS support.

#### Benchmarking Results

Here are some of the results of running the benchmarking commands on my system:

- Emulate old element retrieval requires 66 milliseconds to find 62 structural column elements, whereas the new filtered element collector takes 10 milliseconds for the same task.- Using System.Linq.IEnumerable.OfType<> to extract 142 mullions out of a collection of 346 family instances takes less than one millisecond, whereas looping through all 346 instances and checking them one by one using cast takes 19 milliseconds.- 10 versus 22 versus 35 milliseconds to extract 52 family instances on a given level using ElementLevelFilter, ElementParameterFilter and LINQ, respectively.- Regeneration Example 3: 109 milliseconds to create four walls and regenerating after every addition versus 84 milliseconds doing it once only after add finished.- Regeneration Example 4: 188 milliseconds to update parameter and regenerate at each change versus 80 milliseconds to do it only once at the end.

Here are some further notes on the samples and benchmarking results:

#### Revit 2011 API versus 2010 Performance Comparison

As said,
the old element retrieval emulation requires 66 milliseconds to find 62 structural column elements on my system,
whereas the new filtered element collector takes 10 milliseconds for the same task.

We can only create a very rough performance comparison between Revit 2010 and 2011, since the real 2010 get\_Elements functionality is no longer available. We select all family instances in the model, iterate over them and check their category against the built-in category for structural columns to simulate the 2010 marshalled access and managed code filtering.

In 2011, the same elements can be retrieved by using the OfClass and OfCategory shortcuts for the class and category filters. This is not an exact comparison, because we still have to start off with a collector in both cases. The main point is that access to the data to find a match is much slower in explicit code than using a filter for the same data, where matching is happening natively and only results are returned to you.

#### Structural Material, Usage and Inversion

To select all real structural elements, we select elements whose structural material type is Steel. We are interested in all stick elements, i.e. many different element types like brace, column, girder, etc. Since there are fewer types that we are not interested in, it makes sense to use an inverted filter. We therefore set up four inverted filters for structural instance usage requiring the passing element NOT to be Automatic, Other, Undefined, or Wall. The resulting total of five filters are added to a list and passed in to a logical AND filter to combine all five.

#### Bounding box intersection, view id and exclusion

I already discussed the filter example 2 command implemented by FilterEx2 to
[find all neighbouring elements of a selected one](http://thebuildingcoder.typepad.com/blog/2010/12/find-intersecting-elements.html) in
a separate post.
It uses the quick BoundingBoxIntersectsFilter to find all elements that intersect the selected element, the collector constructor taking a view id to narrow down to only viewable elements, and finally an exclusion filter to eliminate known elements to exclude such as the picked element itself and the view.
It can be run in the StructuralUsage.rvt sample model.

This example finds all related neighbouring elements without even looking at any geometry, very cool.

The bounding box filtering classes include checks for contained within, point inside, bounding box inside, and bounding box intersect.

#### LINQ

After making maximum use of the built-in filtered element collector functionality, we can often make use of LINQ to further hone in, narrow down, and post-process the results. The sample code includes the following three examples which illustrate this:

1. Further filter information using a query- Fast filter for non-native classes- Create an XML report of family and instance usage

**1.** Further filter information using a query:
The first of these addresses the task of finding all furniture family instances which include a material of a given name.
From the material name, we retrieve the material instance.
A filtered element collector returns all family instances of the furniture category.
We then use LINQ to select the elements whose material contains the required material.
This sample is intended to be run in the simple house model.

**2.** Fast filter for non-native classes:
One of the few tasks that cannot be solved with the filtered element collectors alone is retrieval of non-native classes.
Some of the classes available in the Revit API have been added for the sake of developer's convenience and are not present in the internal native Revit code, such as the mullion class, so they cannot be filtered for directly.
For a full list of the 15 non-native classes that cannot be filtered for, please refer to the reference documentation for ElementClassFilter.
I discussed a couple of other examples in the past, e.g. the
[AnnotationSymbol](http://thebuildingcoder.typepad.com/blog/2010/08/filtering-for-a-nonnative-class.html) and
[Panel](http://thebuildingcoder.typepad.com/blog/2010/10/measurepanelarea-update.html) classes.

LINQ provides a handy way to filter for these non-native classes.
We can use a filtered element collector to return all family instances, the mullion parent class, then search for real mullions.
This search is implemented twice for benchmarking purposes, once using the generic LINQ OfType<> predicate, and again using explicit cast.
OfType is much faster.
In this case, in the ArchSample.rvt model file, we have 346 instances, whereof 142 are mullions.
The cast takes more than 5 ms, OfType less than 1.

Kevin also presents some code to use LINQ to hone down even further, to a specific type of mullion, the thin horizontal type, of which there are 14 instances, and which was specifically modified in the sample project to show a different mullion type.

This shows that LINQ is useful to find things that you cannot normally filter on, as well as narrowing down results.

**3.** XML report of family and instance usage:
Besides filtering and post-processing results, LINQ and the Xml.Linq namespace provide a lot of other functionality, such as the ability to rapidly and efficiently create an XML report.
For this family and instance usage report, we retrieve all the families in the project, for each family get all symbols, for each symbol all instances, and report the entire result to XML.
Here LINQ provides a very powerful way of manipulating data very fast.

This sample demonstrates that a lot of information can be extracted with little coding and much support from LINQ.
It uses one collector to retrieve all families in the document, retrieves all instances of each, and writes a structured tree report to XML.
For more details, please refer to the recent
[detailed discussion](http://thebuildingcoder.typepad.com/blog/2010/12/xml-family-usage-report.html) of this.

#### Revit Filters versus LINQ

The parameter filter example 2 implemented by FilterParameterEx2 is of special interest, since it benchmarks and compares the performance of the built-in Revit filtering functionality provided by the level and parameter filters with the performance achieved by post-processing the filter results using LINQ.
Happily,
[the built-in filters do prove to be more efficient](http://thebuildingcoder.typepad.com/blog/2010/10/level-filter-benchmark.html).

Kevin's powerful collection of samples shows that significant enhancements are possible using appropriate filters, regeneration options and transaction modes, and that it pays off to compare various alternatives.