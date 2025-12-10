---
post_number: "0908"
title: "LoadFamily and Collector Iteration Performance"
slug: "load_family_perform"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'revit-api', 'sheets', 'views', 'walls']
source_file: "0908_load_family_perform.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0908_load_family_perform.html"
---

### LoadFamily and Collector Iteration Performance

Since we are into the topic of family loading performance anyway, after yesterdays discussion on
[efficient sweep creation](http://thebuildingcoder.typepad.com/blog/2013/03/sweep-family-performance-enhancement.html),
here is another interesting discussion with some useful family loading and symbol filtering benchmarking results and observations by Thomas Jarzyna and an in-depth analysis of the effects of converting a filtered element collector to a .NET collection by Scott Conover:

**Question:** I need to load some family symbols into my project.
I have a lot of different families and even more symbols, so to keep the projects small and clean I would prefer to load family symbols on demand only.

Therefore, whenever an instance of a family is to be created, I need to check first whether the family symbol is already loaded.

I can see two possible approaches to achieve this:

- Check whether the family symbol already exist by iterating through the database using a filtered element collector searching for family name and family symbol name.
- Not checking anything, and just always calling LoadFamilySymbol. It runs fine even if the family symbol is already loaded. One negative side effect of this approach is the "one time conversion" dialogue popping up for older families every time.

Which approach is better from a performance centred point of view, please?

Is there any other possible approach?

**Answer:** Both of your two approaches sound feasible.

I also find the "one time conversion" dialogue rather irritating and would not want my users to see it, especially not repeatedly.

If that is a serious problem for you, you could implement a system to ensure that it never occurs more than once. For instance, you could create different sub-directories for the different Revit versions of the same family, and check whether an updated version has already been generated before loading an older one.

However, I would expect the use of the filtered element collector to be much (very much!) more efficient.

However^2, it really depends on so many factors that it is impossible to say for sure.

The only way to really answer this with certainty for your particular system, workflow needs, and installation situation is to benchmark the different options.

I described a
[simple benchmarking system](http://thebuildingcoder.typepad.com/blog/2012/01/timer-code-for-benchmarking.html) that
I would use in a Revit add-in in lack of anything more sophisticated.

I cannot think of any other approach offhand.

The thought of calling LoadFamily without even checking beforehand actually never occurred to me :-)

Here are two different implementation examples checking whether a family has already been loaded, and loading it otherwise, for

- [Creating and inserting an extrusion family](http://thebuildingcoder.typepad.com/blog/2011/06/creating-and-inserting-an-extrusion-family.html#3)
- [Structural concrete setout points](http://thebuildingcoder.typepad.com/blog/2012/08/structural-concrete-setout-point-add-in.html#6)

I hope they fulfil your needs.

**Response:** Thank you for the answer.

I was hoping for a simple standard best practice, but I see it's not as easy as that.

In the meantime, I've done a little bit of benchmarking myself.
Just basic, using a stopwatch:

![Filter for and load family benchmarks](img/RevitInsertion.png)

And you are right, depending on the quantity of different Types (FamilySymbols) these approaches differ significantly.

For a scenario of creating 2000 family instances from one family (see attached sheet), this is my conclusion:

- If you expect less than a dozen different family symbols to load, just load them once via "LoadFamilySymbol" and
  don't care about iterating the element collection for the already loaded family symbols.- Otherwise, iterating and finding the loaded family symbols first is better.- Of course, this depends on the complexity of the family types and the environment (hardware & current load).- A large number of different families is not considered yet.

I also noted the following interesting fact while iterating for "known" family symbols:
```csharp
var collector = new FilteredElementCollector( doc );
IEnumerable<FamilySymbol> familySymbols = collector
.OfClass( typeof( FamilySymbol ) )
.ToElements()
.OfType<FamilySymbol>();
```

The additional call to the ToElements method makes the search using the collector slightly less effective (some milliseconds slower) compared to:
```csharp
IEnumerable<FamilySymbol> familySymbols = collector
.OfClass( typeof( FamilySymbol ) )
.OfType<FamilySymbol>();
```

But in the long term, iterating over the family symbols is much faster when ToElements is called first.

For the 2000 instances it takes 14 seconds without ToElements vs. 9 seconds with.

I guess this has to do with the caching provided by
[possible multiple enumerations of IEnumerable](http://confluence.jetbrains.com/display/ReSharper/Possible+multiple+enumeration+of+IEnumerable).

What do you think?

Thank you, till next time.

**Answer:** Thank you for your very interesting results and observations, and congratulations on your first benchmarking results!
They are already very interesting.

A call to ToElements definitely costs some performance, since it allocates an entirely new generic .NET collection and copies all the elements across from the Revit filtered element collector, as I mentioned discussing
[FindElement and collector optimisation](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html).

I am surprised and interested (and was initially a little disappointed) to hear that the subsequent iteration over the .NET collection is measurably faster than the Revit collector, though.

Yes, the speed improvement is probably indeed related to the explanation you point to.
In this case, the Revit element collector may be storing the elements in 'Revit memory' and eating cycles when marshalling them across to '.NET memory'.
This marshalling effort may need to be performed anew on each repetition of the iteration.

Ah yes, with that explanation, I am perfectly happy with this observation, and not disappointed at all.

Thank you for these illuminating results.

**Scott adds:** I would expect the total time spent in FilteredElementCollector.ToElements and subsequent iteration to be longer than direct iteration of FilteredElementCollector, regardless of the number of elements that pass the filter.

Of course, once ToElements has run and you store the managed handles, iteration of this collection should be fast – you are not going back to the document to find the elements again or filter them or wrap them in managed handles.

If you are doing more than one iteration of a given filter criteria, the first one will always be faster than the second – because of the extra time taken to create the Elements collection. The other activities – traversing the database, applying and evaluating the filter, wrapping the elements that pass into managed code – happen on both.

If you know you need to have the same filtered set of elements more than once, saving them in the collection should be faster for the second and subsequent iterations. This assumes that you know the document hasn’t changed where there are new elements that pass the filter (or some of them are deleted). The second iteration over the managed collection should be faster than re-traversing the document. This is a pretty specialized approach. The other reasons for converting FilteredElementCollector to a collection (you want to store a list of elements or ids for later, you need such a list as input to another method, you want to delete anything you find, etc.) still apply.

If you are running multiple loops on the collector with no changes in between, the second will be consistently faster.
Optimally, you might want to unroll the loop to just do everything in one shot.

The first might be faster than the second if the collector is returning a really large number of elements.

Mostly, you want to get a set of things once and just do things to it once.
Normally, only certain specialized applications such as multiple data extractions or multiple permutations for design studies might require retrieval of the exact same data and then doing things to it repeatedly in the same code sequence.

Let’s use made up concrete numbers.

- It takes 100 ms to process the model and find elements that pass the filters, and to wrap each element into the API object.- It takes 5 ms to build the .NET collection returned by ToElements.- It takes 1 ms to traverse a .NET IEnumerable<Element>, which could be either the Revit collector or the .NET collection.

So with one group of elements processed once:

- With collection: 100 + 5 + 1 = 106 ms.- Without collection: 100 + 1 = 101 ms.

With the same exact group of elements processed 10 times in the same code (where we know there is no chance new elements were added or elements deleted that we care about):

- With collection obtained once: 100 + 5 + 10 \* 1 = 116 ms.- Without collection: 100 \* 10 + 10 \* 1 = 1010 ms.

Make sense?

The key thing to understand is that FilteredElementCollector caches nothing (except the filters applied to it, of course).
Caching data needed for multiple things will be better – always – but premature if there’s no need to have such a cache at all.

Note that the built-in collections on Document (.WallTypes, etc.) – which are becoming obsolete in future versions – use FilteredElementCollector as their base; there is no caching done there either, and no real advantage to them other than saving the coding effort of building the collector and filters.