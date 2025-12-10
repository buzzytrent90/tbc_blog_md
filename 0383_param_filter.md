---
post_number: "0383"
title: "Parameter Filter"
slug: "param_filter"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'python', 'references', 'revit-api', 'rooms', 'transactions', 'views', 'walls', 'windows']
source_file: "0383_param_filter.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0383_param_filter.html"
---

### Parameter Filter

AEC DevCamp begins here in Boston today.
I started writing this on Sunday evening, before it begins, waiting in the lobby for my colleagues.

Later on: finishing it off after a nice dinner with Mikako, Saikat, Partha amd Miro...

Later still: posting it first thing in the morning before breakfast...

One area that several people have recently been struggling with is the use of parameter filters.
As shown by the
[filtered element collector benchmarking](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html),
the use of a parameter filter is significantly more efficient than manually postprocessing the results, even though the parameter filter is a slow filter, as opposed to quick filters such as OfClass and OfCategory.

Of course, you should always apply as many quick filters as possible first to narrow down the set of candidate elements before applying any slow or manual filtering.

We already looked at a few parameter filtering examples in the past, for instance to retrieve specific elements based on their
[element name](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) in
the filtered element collector benchmark, their
[element id](http://thebuildingcoder.typepad.com/blog/2010/04/retrieving-newly-created-elements-in-revit-2011.html),
and stair elements
[on a specific level](http://thebuildingcoder.typepad.com/blog/2010/04/retrieve-stairs-on-level.html).
Below are some other samples showing different ways to use parameter filters:

- [Retrieving beams on a specific level](#1).- [Retrieving elements in a specific room](#2).- [Retrieving levels by name](#3).- [ElementId parameter](#4).- [Boolean parameter](#5).- [Double parameter](#6).- [String parameter](#7).

In hindsight, and analysing the differences and similarities between the different examples below, you will notice that it all boils down to ensuring that you know exactly what values you are looking for and which built-in parameters can be used to determine them.
In some cases, there may be exceptions and surprises lying in wait there for you, such as different built-in parameters being used to hold analogous data on different elements...

#### Retrieving beams on a specific level

**Question:** My model contains various elements such as walls, beams, slabs and columns on Level 1, and I am trying to retrieve them all using an ElementLevelFilter like this:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  ICollection<Element> levels
    = collector.OfClass( typeof( Level ) )
      .ToElements();

  for( int i = 0; i < levels.Count; i++ )
  {
    ElementId levelId = levels.ElementAt( i ).Id;

    ElementLevelFilter levelFilter
      = new ElementLevelFilter( levelId );

    collector = new FilteredElementCollector( doc );

    ICollection<Element> allOnLevel
      = collector.WherePasses( levelFilter )
        .ToElements();

    // . . .
  }
```

I am expecting all elements on Level 1 to be returned in this collection, but in fact I see that while the walls, slabs and columns are present, all the beam elements are missing.

Why are the beams not included?

How can I set up the filtered element collector to retrieve them as well?

**Answer:** The elements returned by your collector are of wall, column and slab type.
These elements all have a valid value in their Level property.

However, the beams' Level property value is null, because the beam is not a level based component.
A beam does not have a Base Level or Base Constraint parameter.

It does have a Reference Level parameter, however.
You can therefore retrieve beams on a specific Level using a parameter filter that checks its reference level.

Here is a code snippet to set up a filtered element collector and an appropriate parameter filter.
You can add the beams returned by this to the original collection:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfCategory(
    BuiltInCategory.OST\_StructuralFraming );

  collector.OfClass( typeof( FamilyInstance ) );

  BuiltInParameter bip = BuiltInParameter
    .INSTANCE\_REFERENCE\_LEVEL\_PARAM;

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId( bip ) );

  FilterNumericRuleEvaluator evaluator
    = new FilterNumericGreater();

  ElementId idRuleValue = level.Id;

  FilterElementIdRule rule
    = new FilterElementIdRule(
      provider, evaluator, idRuleValue );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  collector.WherePasses( filter );
```

#### Retrieving Elements in a Specific Room

**Question:** I have been able to scan a model manually to select elements, such as Furniture, and determine which room each item is in.
I would like to efficiently find all the elements that are associated with a specific room, without manually scanning through all the elements in the model.
Is there a structure in the API that will find me a list of elements that are in a room?

**Answer:** The Revit 2011 API filtered element collector functionality includes parameter filters which can be used to conveniently retrieve all components located in a specified room.
The parameter filter can be used because the room location is available through the built-in parameter ELEM\_ROOM\_NUMBER.
Here is a complete external command example demonstrating this:
```python
[TransactionAttribute( TransactionMode.ReadOnly )]
[RegenerationAttribute( RegenerationOption.Manual )]
public class RevitCommand : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    ElementId id = new ElementId(
      BuiltInParameter.ELEM\_ROOM\_NUMBER );

    ParameterValueProvider provider
      = new ParameterValueProvider( id );

    FilterStringRuleEvaluator evaluator
      = new FilterStringEquals();

    string sRoomNumber = "1";

    FilterRule rule = new FilterStringRule(
      provider, evaluator, sRoomNumber, false );

    ElementParameterFilter filter
      = new ElementParameterFilter( rule );

    FilteredElementCollector collector
      = new FilteredElementCollector( doc )
        .WherePasses( filter );

    string s = string.Empty;

    foreach( Element elem in collector )
    {
      s += elem.Name + elem.Category.Name.ToString() + "\n";

    }
    System.Windows.Forms.MessageBox.Show( s );

    return Result.Succeeded;
  }
}
```

#### Retrieving levels by name

As mentioned above, we already looked at examples of retrieving
[elements by element name](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) and
[stairs on a specific level](http://thebuildingcoder.typepad.com/blog/2010/04/retrieve-stairs-on-level.html).
Here is a slightly different issue, retrieving levels by name, encountered by Kevin Vandecar:

**Question:** I am covering the parameter filter in my DevCamp presentation and I have to admit I have not used it before.
I checked out the
[benchmarking code](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) which
uses a parameter filter to find level by name.
However, when I ran it, it did not work for me and GetFirstNamedElementOfTypeUsingParameterFilter always returned null.
I then moved on to trying some other examples, which are appended below.
I got them working ok...

**Answer:** So then I went back to the benchmarking code to see why I was not getting anything back.
It took some debugging, but for whatever reason, it appears that a Level object stores its Name parameter in the built-in parameter DATUM\_TEXT parameter instead of ELEM\_NAME\_PARAM.

That leads me to another question:
How does an API user typically figure out which parameter to use?
If I use RevitLookup on Level, I can also see a parameter named 'Name' there, but...

WOW, hold on!
I just tried RevitLookup again, and it does show the built-in parameter API name as well!
COOL!
I clicked Level, selected a level, then clicked Parameters, then selected Name, then Definition, and then the dialog shows an entry: Built in Param = DATUM\_TEXT.
So I guess that confirms it.

Wow, I wish I had known that last night!

Anyway, here
s the final code I used to test to find all Levels containing the string "Level" in their Name parameter, i.e. in the built-in parameter DATUM\_TEXT:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  collector.OfClass( typeof( Level ) );
  ElementId id = new ElementId(
    BuiltInParameter.DATUM\_TEXT );

  ParameterValueProvider provider
    = new ParameterValueProvider( id );

  FilterStringRuleEvaluator evaluator
    = new FilterStringContains();

  FilterRule rule = new FilterStringRule(
    provider, evaluator, "Level", false );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );
```

Ok, I am on my way again, and hope this bit of research helps.

Here are the other examples that Kevin mentioned:

#### ElementId Parameter

Use numeric evaluator and integer rule to test ElementId parameter;
Filter levels whose id is greater than specified id value:
```csharp
  BuiltInParameter testParam
    = BuiltInParameter.ID\_PARAM;

  ParameterValueProvider pvp
    = new ParameterValueProvider(
      new ElementId( ( int ) testParam ) );

  FilterNumericRuleEvaluator fnrv
    = new FilterNumericGreater();

  // filter elements whose Id is greater than 99

  ElementId ruleValId = new ElementId( 99 );

  FilterRule paramFr = new FilterElementIdRule(
    pvp, fnrv, ruleValId );

  ElementParameterFilter epf
    = new ElementParameterFilter( paramFr );

  FilteredElementCollector collector
    = new FilteredElementCollector(  doc );

  collector.OfClass( typeof( ViewPlan ) )
    .WherePasses( epf ); // only deal with ViewPlan
```

#### Boolean Parameter

Use numeric evaluator and integer rule to test Boolean parameter;
Filter levels whose crop view is false:
```csharp
  int ruleValInt = 0;

  testParam = BuiltInParameter.VIEWER\_CROP\_REGION;

  pvp = new ParameterValueProvider(
    new ElementId( ( int ) testParam ) );

  fnrv = new FilterNumericEquals();

  paramFr = new FilterIntegerRule(
    pvp, fnrv, ruleValInt );

  epf = new ElementParameterFilter( paramFr );

  collector = new FilteredElementCollector( doc );

  collector.OfClass( typeof( ViewPlan ) )
    .WherePasses( epf ); // only deal with ViewPlan
```

#### Double Parameter

Use numeric evaluator and double rule to test double parameter;
Filter levels whose top offset is greater than specified value:
```csharp
  double ruleValDb = 10;

  testParam =
    BuiltInParameter.VIEWER\_BOUND\_OFFSET\_TOP;

  pvp = new ParameterValueProvider(
    new ElementId( ( int ) testParam ) );

  fnrv = new FilterNumericGreater();

  paramFr = new FilterDoubleRule(
    pvp, fnrv, ruleValDb, double.Epsilon );

  epf = new ElementParameterFilter( paramFr );

  collector = new FilteredElementCollector( doc );

  collector.OfClass( typeof( ViewPlan ) )
    .WherePasses( epf ); // only deal with ViewPlan
```

#### String Parameter

Use string evaluator and string rule to test string parameter;
Filter all elements whose view name contains level:
```csharp
  String ruleValStr = "Level";

  testParam = BuiltInParameter.VIEW\_NAME;

  pvp = new ParameterValueProvider(
    new ElementId( ( int ) testParam ) );

  FilterStringRuleEvaluator fnrvStr
    = new FilterStringContains();

  paramFr = new FilterStringRule(
    pvp, fnrvStr, ruleValStr, false );

  epf = new ElementParameterFilter( paramFr );

  collector = new FilteredElementCollector( doc );

  collector.OfClass( typeof( ViewPlan ) )
    .WherePasses( epf ); // only deal with ViewPlan
```

Many thanks to Joe Ye and Kevin Vandecar for these new examples!

I hope we have collected and presented enough parameter filter samples now to cover all possible cases!
If not, I would not be surprised, and just let me know, please...