---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: code_example
optimization_date: '2025-12-11T11:44:16.635733'
original_url: https://thebuildingcoder.typepad.com/blog/1736_faster_filtering.html
post_number: '1736'
reading_time_minutes: 6
series: filtering
slug: faster_filtering
source_file: 1736_faster_filtering.md
tags:
- elements
- family
- filtering
- parameters
- python
- revit-api
- sheets
- views
title: Faster Filtering
word_count: 1237
---

### Slow, Slower Still and Faster Filtering
Today I discuss (once again) an important performance aspect of Revit element filtering, a Python script for tagging JPEG images with EXIF data, prompted by a recent ski tour, and three other interesting topics that caught my eye:
- [Slow, slower still and faster filtering](#2)
- [Python JPEG EXIT filename tagging](#3)
- [TED talks and population growth](#4)
- [Objective reality does not exist](#5)
- [Artificial intelligence judge](#6)
#### Slow, Slower Still and Faster Filtering
A question was raised in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
asking, [is it slow filter](https://forums.autodesk.com/t5/revit-api-forum/is-it-show-filter/m-p/8712175)?
Meseems we need to point out yet again the difference between slow and fast filters.
Much more importantly, though, the difference between using Revit filters or post-processing results using .NET and or LINQ.
The latter will always be at least twice as slow as using Revit filters, regardless of whether they are fast or slow.
The process of marshalling the Revit element data out of the Revit memory space into the .NET environment always costs much more time than retrieving the data inside of Revit in the first place.
Therefore, if you care about performance, use as many Revit filters as possible, regardless whether they are fast or slow.
In the following example, .NET post-processing is used to test a parameter value.
That can be speeded up by an order of magnitude by implementing a Revit parameter filter instead, which avoids the marshalling, post-processing, and is fast to boot:
\*\*Question:\*\* I have used the following code:
```csharp
string section_name = "XXXXXXX";
IEnumerable elems = collect
.OfClass(typeof(FamilyInstance))
.Where(x => x.get_Parameter(
BuiltInParameter.ELEM_FAMILY_PARAM)
.AsValueString() == section_name );
```
Is it a slow filter?
\*\*Answer:\*\* No, it is not. It is even slower than a slow filter.
Your code retrieves the element data from Revit to the .NET add-in memory space, and then uses .NET and LINQ to post-process it.
The difference is explained in the discussion
of [quick, slow and LINQ element filtering](http://thebuildingcoder.typepad.com/blog/2015/12/quick-slow-and-linq-element-filtering.html).
You could convert it to a fast filter by implementing
a [parameter filter to compare the family name](https://thebuildingcoder.typepad.com/blog/2018/06/forge-tutorials-and-filtering-for-a-parameter-value.html#3).
That discussion does not show how to actually implement the parameter filter.
I therefore cleaned up The Building Coder sample code
demonstrating [retrieving named family symbols using either LINQ or a parameter filter](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs#L1203-L1255) for
you to illustrate how to do that:
```csharp
#region Retrieve named family symbols using either LINQ or a parameter filter
static FilteredElementCollector
GetStructuralColumnSymbolCollector(
Document doc )
{
return new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_StructuralColumns )
.OfClass( typeof( FamilySymbol ) );
}
static IEnumerable Linq(
Document doc,
string familySymbolName )
{
return GetStructuralColumnSymbolCollector( doc )
.Where( x => x.Name == familySymbolName );
}
static IEnumerable Linq2(
Document doc,
string familySymbolName )
{
return GetStructuralColumnSymbolCollector( doc )
.Where( x => x.get_Parameter(
BuiltInParameter.SYMBOL_NAME_PARAM )
.AsString() == familySymbolName );
}
static IEnumerable FilterRule(
Document doc,
string familySymbolName )
{
return GetStructuralColumnSymbolCollector( doc )
.WherePasses(
new ElementParameterFilter(
new FilterStringRule(
new ParameterValueProvider(
new ElementId( BuiltInParameter.SYMBOL_NAME_PARAM ) ),
new FilterStringEquals(), familySymbolName, true ) ) );
}
static IEnumerable Factory(
Document doc,
string familySymbolName )
{
return GetStructuralColumnSymbolCollector( doc )
.WherePasses(
new ElementParameterFilter(
ParameterFilterRuleFactory.CreateEqualsRule(
new ElementId( BuiltInParameter.SYMBOL_NAME_PARAM ),
familySymbolName, true ) ) );
}
#endregion // Retrieve named family symbols using either LINQ or a parameter filter
```
It demonstrates two ways to implement the highly efficient parameter filter.
First, the explicit solution, implementing the separate `ElementParameterFilter`, `FilterStringRule`, and `ParameterValueProvider` components one by one.
Second, a much more succinct solution using the `ParameterFilterRuleFactory` `CreateEqualsRule` method.
Here are some other examples of using the `ParameterFilterRuleFactory`:
- [Isolating elements of a given system](https://thebuildingcoder.typepad.com/blog/2015/01/isolating-elements-of-a-given-system.html)
- [Retrieving elements visible in view](https://thebuildingcoder.typepad.com/blog/2017/05/retrieving-elements-visible-in-view.html)
- [A new way to retrieve a parameter id](https://thebuildingcoder.typepad.com/blog/2018/08/revit-20191-cefsharp-forge-accelerator-in-rome.html#7)
I hope this clarifies.
Now for some non-Revit-related topics:
#### Python JPEG EXIT Filename Tagging
I went on a ski tour in the Silvretta area with some friends two weeks ago, climbing Haagspitze, Hintere Jamspitze, Ochsenkopf, Sattelkopf and above all Piz Buin.
Here I am crossing the Piz Buin summit ridge:
![Jeremy on the Piz Buin summit ridge](img/20190323_121516_jk_8144_jeremy_on_ridge_1024x768.jpg)
That excursion prompted me to implement a Python script to tag the photos shared by the participants after the tour.
How to easily sort all the different pictures chronologically?
Well, most cameras embed a timestamp in EXIF data when they generate a JPEG image.
This information can be easily accessed and used to rename each image file, e.g., prepending the timestamp to the filename.
In case of multiple photographers, it also makes sense to add the photographer initials to the filename.
I implemented a really minute little Python script `ren2timestamp.py` that does this for me, using a hint from the StackOverflow discussion
on [getting date and time when photo was taken from EXIF data using PIL]( https://stackoverflow.com/questions/23064549/get-date-and-time-when-photo-was-taken-from-exif-data-using-pil) and
the [`exif-py` easy-to-use Python module to extract Exif metadata from tiff and jpeg files](https://github.com/ianare/exif-py):

```
# !/usr/bin/env python
# ren2timestamp.py - rename files to add their exif timestamp as a prefix
# Copyright (C) 2018 by Jeremy Tammik, Autodesk Inc.

import exifread, datetime, os, sys, time

files = sys.argv[1:]

exif_ts_key = 'EXIF DateTimeOriginal'

def exif_timestamp(filename):
  with open(filename, 'rb') as fh:
    tags = exifread.process_file(fh, stop_tag=exif_ts_key)
    if tags.has_key(exif_ts_key):
      return tags[exif_ts_key]
    else:
      return ''

photographer_initials = 'jt'

for g in files:
  ts3 = exif_timestamp(g)
  ts3 = str(ts3).replace(':','').replace(' ','_')
  newname = ts3 + '_' + photographer_initials + '_' + g
  print("%s --> '%s'" % (g,newname))
  os.rename(g, newname)
```

Here is the closing picture of the entire group on the Sattelkopf summit:
![Group photo on Sattelkopf](img/20190324_123346_wk_1091_gruppenfoto_sattelkopf_912x606.jpg)
#### TED Talks and Population Growth
[TED](http://www.ted.com) is a non-profit organisation devoted to spreading ideas, usually in the form of short, powerful talks (18 minutes or less).
TED is also a global community, welcoming people from every discipline and culture who seek a deeper understanding of the world.
TED believes passionately in the power of ideas to change attitudes, lives and, ultimately, the world.
TED.com is a clearinghouse of free knowledge from the world's most inspired thinkers.
I have listened to many TED talks, enjoyed and appreciated them very much, and highlighted several of them here in the past.
Here is an illuminating one with a paradoxical answer by [Hans Rosling on Global population growth, box by box](https://youtu.be/fTznEIZRkLg):

The world's population will grow to 9 billion over the next 50 years – and only by raising the living standards of the poorest can we check population growth.
#### Objective Reality Does Not Exist
[A quantum experiment suggests there’s no such thing as objective reality](https://www.technologyreview.com/s/613092/a-quantum-experiment-suggests-theres-no-such-thing-as-objective-reality).
#### Artificial Intelligence Judge
[Can AI be a fair judge in court? Estonia thinks so](https://www.wired.com/story/can-ai-be-fair-judge-court-estonia-thinks-so).