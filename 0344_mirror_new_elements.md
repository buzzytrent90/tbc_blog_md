---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 10.9
content_type: code_example
optimization_date: '2025-12-11T11:44:13.779857'
original_url: https://thebuildingcoder.typepad.com/blog/0344_mirror_new_elements.html
post_number: '0344'
reading_time_minutes: 11
series: elements
slug: mirror_new_elements
source_file: 0344_mirror_new_elements.htm
tags:
- csharp
- elements
- filtering
- geometry
- levels
- parameters
- python
- references
- revit-api
- selection
- transactions
title: Retrieving Newly Created Elements in Revit 2011
word_count: 2248
---

### Retrieving Newly Created Elements in Revit 2011

Here is a largish post which will hopefully keep you happily occupied over the weekend.
We looked at the topic of
[retrieving newly created elements](http://thebuildingcoder.typepad.com/blog/2009/07/retrieving-newly-created-elements.html) in
Revit 2010 last year.
Since the method presented there is based on simply retrieving sequential entries from the Revit element collection, and that collection is no longer directly accessible in Revit 2011, I was curious to see whether the approach presented there would still work.
To my surprise, with some adaptation, it really does.

I also discovered a new approach which I believe to be significantly more reliable.
It is based on the element ids, and the assumption that they will be monotonously increasing, instead of simply using the sequential order of retrieval of the elements themselves.
This new algorithm makes use of the Revit 2011 API filtered element collector mechanism and a parameter filter.
Since my recent
[collector benchmarking](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) seems
to indicate that any kind of parameter filter will be significantly faster than post-processing the collector results, I would assume that this new approach is faster as well.

Please note that this technique is completely unnecessary for actually detecting newly added elements in real life, since the new Revit 2011
[DocumentChanged element level event](http://thebuildingcoder.typepad.com/blog/2010/04/element-level-events.html#1) provides
the application subscribing to it a set of all newly added elements each time a transaction is closed.
So be aware that the following exercise is mainly academic in nature.
Thank you very much, Guy, for noticing and pointing that out below.

Before looking at how to retrieve the newly created elements, we implemented a minimal command demonstrating the
[use of the Document.Mirror method](http://thebuildingcoder.typepad.com/blog/2009/07/mirror-an-element.html) in
Revit 2010 and used the result of that method to test the retrieval algorithm.
I updated that command for Revit 2011 as well, and think it is worthwhile taking a quick look at the differences.

So here are the topics I will be discussing here:

- [Element mirroring and code changes](#1).- [Revit element access](#2).- Newly created element retrieval based on [sequential element order](#3).- Newly created element retrieval based on [monotonously increasing element id values](#4).- [Enhanced parameter filter](#5) for greater element id values.- [Download](#6)

#### Element Mirroring

Let us first have a look at the differences between the 2010 and 2011 versions of the very simple CmdMirror command.
All it does is apply the Document.Mirror method on the set of currently selected elements.
Here is the code for Revit 2010:
```python
class CmdMirror : IExternalCommand
{
  public CmdResult Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    Application app = commandData.Application;
    Document doc = app.ActiveDocument;

    ElementSet els = doc.Selection.Elements;

    Line line = app.Create.NewLine(
      XYZ.Zero, XYZ.BasisX, true );

    doc.Mirror( els, line );

    return CmdResult.Succeeded;
  }
}
```

The equivalent code for Revit 2011 looks like this:
```python
[Transaction( TransactionMode.Automatic )]
[Regeneration( RegenerationOption.Manual )]
class CmdMirror : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;

    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    ElementSet els = uidoc.Selection.Elements;

    Line line = app.Create.NewLine(
      XYZ.Zero, XYZ.BasisX, true );

    doc.Mirror( els, line );

    return Result.Succeeded;
  }
}
```

Note the following differences:

- The class now requires the transaction mode and regeneration options.- The CmdResult enumeration has been renamed.- There are separate classes for the user interface and database versions of the application and document classes.

In addition to the code itself, the Revit namespaces have changed completely, and we also have to reference two Revit API assemblies instead of just one now after the
[DLL split](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-is-coming.html) and
[namespace refactoring](http://redbolts.com/blog/post/2010/03/27/Revit2011-API-e28093-Whate28099s-in-a-name.aspx).
Here are the namespaces and classes that we reference in the Revit 2010 version of the code:
```csharp
using System;
using System.Collections.Generic;
using Autodesk.Revit;
using Line = Autodesk.Revit.Geometry.Line;
using XYZ = Autodesk.Revit.Geometry.XYZ;
using CmdResult = Autodesk.Revit.IExternalCommand.Result;
```

In the Revit 2011 API, we make use of the following namespaces instead:
```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;
```

Other than that, nothing has changed.
In this super-simple seven-line command, though, that does mean that five of the original lines of code have been modified, four more added, and every single Revit namespace reference replaced.

All of the changes we encountered in this case, however, only have to do with the add-in packaging and connection to Revit.
None of them have anything to do with its actual functionality, and the two lines of code which really do any work, the calls to NewLine and Mirror, have in fact not changed at all.
Therefore, in a larger application, you would expect a much smaller percentage of the code to be affected.

Now that we have the mirroring operation sorted out for Revit 2011, let's have a look at the newly created element retrieval algorithm.
But first, a couple of words on Revit element access in general.

#### Revit Element Access

In Revit 2010 and previous versions, the Revit API provided direct access to the Revit element collection.
In the Revit 2011 API, this collection is no longer directly accessible at all, since the API designers have decided that no normal application ever has any legitimate need to iterate over all elements in the database in one go, but should always be interested in only a subset.
To retrieve such subsets, the API provides highly optimised filtered element collectors, as we saw in the recent discussions on
[performance profiling](http://thebuildingcoder.typepad.com/blog/2010/03/performance-profiling.html) and
[collector benchmarking](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html).

In Revit 2010, we implemented a system to retrieve newly added elements, for instance those generated by executing the mirror command, simply by making a note of the total number of elements before executing the command, executing the mirroring operation, and then retrieving all the ones returned after that number when sequentially iterating over the elements collection.
Since the elements collection is no longer accessible in the Revit 2011 API, I was pretty sure that this would no longer work.
Surprisingly, though, it still does.

Since there is no access to the full elements collection, I applied the filter WhereElementIsNotElementType to the document's FilteredElementCollector.
ElementType is 2011-speak for Symbol, i.e. the Revit 2010 class Symbol has been renamed to ElementType in the Revit 2011 API, so this filter returns all elements which are not element types.
I would assume that to include all elements which may result from a mirroring operation.
Please correct me if I am mistaken.

Here is the very simple method making use of this filter, which we will later use for both the sequential element and the monotonously increasing element id retrieval methods:
```csharp
FilteredElementCollector GetElements( Document doc )
{
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );
  return collector.WhereElementIsNotElementType();
}
```

#### Newly Created Element Retrieval Based on Sequential Element Order

To retrieve all elements returned after a given sequence number, similar to the original 2010 algorithm, we implemented the following methods:
```csharp
/// <summary>
/// Return the current number of non-ElementType elements.
/// </summary>
int GetElementCount( Document doc )
{
  return GetElements( doc ).ToElements().Count;
}

/// <summary>
/// Return all database elements after the given number n.
/// </summary>
List<Element> GetElementsAfter( int n, Document doc )
{
  List<Element> a = new List<Element>();
  FilteredElementCollector c = GetElements( doc );
  int i = 0;

  foreach( Element e in c )
  {
    ++i;

    if( n < i )
    {
      a.Add( e );
    }
  }
  return a;
}
```

They are called like this to determine the number of non-ElementType elements before the mirroring operation, execute the mirroring command, and retrieve the newly added elements afterwards:
```csharp
using( SubTransaction t = new SubTransaction( doc ) )
{
  t.Start();

  int n = GetElementCount( doc );

  doc.Mirror( els, line );

  List<Element> a = GetElementsAfter( n, doc );

  string s = \_msg;

  foreach( Element e in a )
  {
    s += string.Format( "\r\n  {0}",
      Util.ElementDescription( e ) );
  }
  Util.InfoMsg( s );

  t.RollBack();
}
```

The entire operation is encapsulated in a sub-transaction which is subsequently rolled back, so that we can immediately continue to test the alternative and more reliable new retrieval method based on element ids instead of the sequence of the elements themselves.

#### Newly Created Element Retrieval Based on Monotonously Increasing Element Id Values

Instead of simply using the sequential ordering of elements as they happen to be returned by the filtered element collector, a much more reliable assumption is that element ids are allocated monotonously increasing values.
Therefore, instead of simply counting the number of elements returned by the GetElements method, as we did above in GetElementCount, a more reliable approach might be to determine the maximal element id value before the mirroring operation, and then retrieve all elements with a higher element id afterwards.

Since the element id is available through a parameter, we can make use of a parameter filter to retrieve these elements.
To make use of a parameter filter, you have to define the following components:

- The provider, e.g. the parameter you are interested in.- The evaluator, e.g. how to compare it with the target value.- The rule, which specifies how the provider and the comparison work together.

We demonstrated one implementation of using a parameter filter in the
[collector benchmark](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) command
CmdCollectorPerformance, in the method GetFirstNamedElementOfTypeUsingParameterFilter, where we used it to find an element with a specific name by checking the built-in parameter ELEM\_NAME\_PARAM for equality with the given target string.

In this case, we set up a parameter filter to retrieve elements whose element id is greater than a given value.
We can achieve this by specifying that the value of the built-in parameter ID\_PARAM must exceed the target element id, like this:
```csharp
FilteredElementCollector GetElementsAfter(
  Document doc,
  ElementId lastId )
{
  BuiltInParameter bip = BuiltInParameter.ID\_PARAM;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  FilterNumericRuleEvaluator evaluator
    = new FilterNumericGreater();

  FilterRule rule = new FilterElementIdRule(
    provider, evaluator, lastId );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  return collector.WherePasses( filter );
}
```

This method returns all elements in the entire document whose element id is greater than 'lastId'.

Here is the code making use of this method, analogously to the snippet above, i.e. determine the maximal element id before the mirroring operation, execute the mirroring command, and retrieve the newly added elements afterwards:
```csharp
using( SubTransaction t = new SubTransaction( doc ) )
{
  t.Start();

  FilteredElementCollector a = GetElements( doc );
  int i = a.Max<Element>( e => e.Id.IntegerValue );
  ElementId maxId = new ElementId( i );

  doc.Mirror( els, line );

  // get all elements in document with an
  // element id greater than maxId:

  a = GetElementsAfter( doc, maxId );

  string s = \_msg;

  foreach( Element e in a )
  {
    s += string.Format( "\r\n  {0}",
      Util.ElementDescription( e ) );
  }
  Util.InfoMsg( s );

  t.RollBack();
}
```

#### Enhanced Parameter Filter for Greater Element Id Values

While writing this, I just noticed some potential for more improvement here.
The parameter filter is a slow filter, so it is important to eliminate as many elements as possible using other quick filters before applying it.
One filter that we did apply in all the previous element retrieval was the non-ElementType one, so why not use it here as well?

A filtered element collector need not necessarily be based on a document, but can also be used to apply further filtering to the results of a previous filter.
Making use of this functionality, we can rewrite our parameter filter method as follows to use a collection of already filtered elements instead of the contents of the entire document:
```csharp
FilteredElementCollector GetElementsAfter(
  FilteredElementCollector input,
  ElementId lastId )
{
  BuiltInParameter bip = BuiltInParameter.ID\_PARAM;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  FilterNumericRuleEvaluator evaluator
    = new FilterNumericGreater();

  FilterRule rule = new FilterElementIdRule(
    provider, evaluator, lastId );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  return input.WherePasses( filter );
}
```

Then we can reuse the GetElements that we used above once again for the parameter filter retrieval as well like this:
```csharp
using( SubTransaction t = new SubTransaction( doc ) )
{
  t.Start();

  FilteredElementCollector a = GetElements( doc );
  int i = a.Max<Element>( e => e.Id.IntegerValue );
  ElementId maxId = new ElementId( i );

  doc.Mirror( els, line );

  // only look at non-ElementType elements
  // instead of all document elements:

  a = GetElements( doc );
  a = GetElementsAfter( a, maxId );

  string s = \_msg;

  foreach( Element e in a )
  {
    s += string.Format( "\r\n  {0}",
      Util.ElementDescription( e ) );
  }
  Util.InfoMsg( s );

  t.RollBack();
}
```

In a real-world application, there will probably be numerous other quick filters that can be applied additionally before resorting to slow ones to eliminate even more uninteresting elements and further improve performance.
It would also be interesting to add some benchmarking code to the three cases presented above and run a test in a reasonably sized model to see whether the performance differences that I am postulating above really exist.
I will leave that as an exercise to the interested reader, for the moment, leaving this post at is, for now, and hope that this exploration proves useful to you.

#### Download

Here is
[version 2011.0.0.64](zip/bc_11_64.zip)
of the complete Visual Studio solution including the updated mirroring and newly created element retrieval commands CmdMirror and CmdMirrorListAdded.