---
post_number: "0426"
title: "Filtering for a Non-Native Class"
slug: "filter_non_native_class"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'revit-api', 'rooms', 'views']
source_file: "0426_filter_non_native_class.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0426_filter_non_native_class.html"
---

### Filtering for a Non-Native Class

I left Ko Tao by boat and night bus to Bangkok and am now heading towards
[Pattaya](http://en.wikipedia.org/wiki/Pattaya),
where my brother Marcus currently lives.
Mentioning this to a resident of Ko Tao I learned that Pattaya is renowned for foreign old age pensioners and prostitutes, and a saying says that "good boys go to heaven – bad boys go to Pattaya".
Marcus has friends there, enjoys the local bridge club, great transport links, a nice apartment at a good price and the best Internet connection he has ever had in Thailand.
I am looking forward to seeing him and it.

I just arrived in Kao San Road in Bangkok at three am in the morning after a good sleep on the bus and now need to find out how to get to Ekamai to catch a bus for the three hour ride over there.

Before continuing, and before returning to the Revit API, here are a few more memorable items from my stay on Ko Tao:

- Maurizio's cocoanut.- My cocoanut.- Ripe mangoes.- A beautiful sunset and new moon.

Maurizio's cocoanut almost fell on and killed him as he sat in a deckchair on the veranda of a bar.
It landed less than a foot away from him and splashed him soaking wet with its juice.
The owners came out, laughed, and said 'Oh yes, that is the second one today'.
They didn't even move the deck chair in case another one might come down and smash it to smithereens.

My cocoanut was a much more pleasurable affair.
I have always wanted to experience step by step how a cocoanut can be opened and consumed.
One morning at breakfast, one fell down ten metres away.
I went and picked it up and asked the workmen next to us how to open it.
They were building a little hut to house a new beach bar, and I thought that their tools might come in useful.
Instead they took me over to a pole stuck firmly in the earth with a sharp long metal blade attached.
You need both hands to hold and twist the nut against the blade to cut and lever off its husk.
Once the husk is off, the point of the same blade can be used to poke out a hole in one of the three weak spots at one end of the nut to stick in a straw to drink the juice.

After I had drunk the juice, I took the empty nut over to the workmen again.
One good whack with a hammer split it cleanly in two, and then I was able to cut out the cocoanut flesh strip by strip all day long.
I didn't finish it until breakfast next morning.

The mangoes are ripe here!
I love ripe mangoes!

I climbed hill and went and sat on a promontory to watch the one sunset that halfway made it through the last days' cloudy skies.
It was a beautiful viewpoint and gave an impressive panorama of several beautiful cloudscapes and storms of all shapes and sizes in all directions.
The sun was obscured before actually setting, but the exquisite thin crescent of the new moon waxing appeared sharp and clear further up soon after.

Anyway, back to the Revit API again, and another episode in the interminable saga of the filtered element collectors, the latest of which were on
[parameter filters](http://thebuildingcoder.typepad.com/blog/2010/06/parameter-filter.html),
a
[correction](http://thebuildingcoder.typepad.com/blog/2010/06/element-name-parameter-filter-correction.html) to that,
and
[filtering for shared parameters](http://thebuildingcoder.typepad.com/blog/2010/08/elementparameterfilter-with-a-shared-parameter.html).
The first of these includes a list of pointers to many previous filtered element collector examples.

Here is another filtering query that has cropped up repeatedly, in spite of the API documentation explaining the situation pretty clearly:

**Question:** Please advise me on how to access AnnotationSymbols in the API.
There is not enough documentation, and no code samples provided in the SDK.

I want to get a collection of annotation symbols in a family document (but this should work on any document)

I first tried to create a filtered element collector
```csharp
ElementClassFilter filter1
= new ElementClassFilter(
typeof( AnnotationSymbol ) );
```

This throws the following exception:
"Input type is of an element type that exists in the API, but not in Revit's native object model. Try using Autodesk.Revit.DB.FamilyInstance instead, and then post-processing the results to find the elements of interest."

After that I tried the following:
```csharp
List ann
= new List();
ElementClassFilter filter1
= new ElementClassFilter(
typeof( FamilyInstance ) );
FilteredElementCollector coll1
= new FilteredElementCollector( doc );
coll1.WherePasses( filter1 );
foreach( Element el in coll1 )
{
AnnotationSymbol anns
= el as AnnotationSymbol;
if( anns != null )
{
ann.Add( el as AnnotationSymbol );
}
}
```

This returns nothing.

I found the post on
[imports in families](http://thebuildingcoder.typepad.com/blog/2009/05/imports-in-families.html),
but the following code provided there does not work in the Revit 2011 API:
```csharp
List annotations = new List();
doc.get\_Elements( typeof( AnnotationSymbol ),
annotations );
```

**Answer:** The message that you received is actually pretty self explanatory: the AnnotationSymbol class is available in the Revit API, but does not correspond to a "real" internal Revit class, so the internal Revit filtering mechanisms cannot search for it.
Instead, you can specify a base class of AnnotationSymbol in the filter, such as the FamilyInstance class.

The code that you found in the blog post calling the get\_Elements method is making use of the document Elements property which returned an iterator over all the database elements in the Revit 2010 API.
That property was removed, because it is much less efficient than the filtered element collector access introduced in the Revit 2011 API.

The Revit API help file RevitAPI.chm lists the AnnotationSymbol base classes:

```
System.Object
  Autodesk.Revit.DB.Element
    Autodesk.Revit.DB.Instance
      Autodesk.Revit.DB.FamilyInstance
        Autodesk.Revit.DB.AnnotationSymbol
```

Here is an example of a helper method that retrieves Revit MEP spaces and runs into the same issue:
```csharp
/// <summary>
/// Retrieve all spaces in given document.
/// </summary>
public static List<Space> GetSpaces( Document doc )
{
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  // trying to collect all spaces directly causes
  // the following error:
  //
  // Input type is of an element type that exists
  // in the API, but not in Revit's native object
  // model. Try using Autodesk.Revit.DB.Enclosure
  // instead, and then postprocessing the results
  // to find the elements of interest.
  //
  //collector.OfClass( typeof( Space ) );

  collector.OfClass( typeof( Enclosure ) );

  return (from e in collector.ToElements()
          where e is Space select e as Space)
    .ToList<Space>();
}
```

The same issue also occurs with rooms, represented by the Room class, also derived from the Enclosure base class. For rooms, another solution is also possible using the dedicated RoomFilter filter class:
```csharp
  FilteredElementCollector rooms
    = new FilteredElementCollector( doc );

  RoomFilter filter = new RoomFilter();
  rooms.WherePasses( filter );
```

I beg to differ from your opinion that "There is not enough documentation". If you look at the description of the ElementClassFilter class that you are using in the sample code you provided, it clearly states:

This filter is a quick filter. Quick filters operate only on the ElementRecord, a low-memory class which has a limited interface to read element properties. Elements which are rejected by a quick filter will not be expanded in memory.
This filter will match elements whose class is an exact match to the input class, or elements whose class is derived from the input class.

There is a small subset of Element subclasses in the API which are not supported by this filter. These types exist in the API, but not in Revit's native object model, which means that this filter doesn't support them. In order to use a class filter to find elements of these types, it is necessary to use a higher level class and then process the results further to find elements matching only the subtype. The following types are affected by this restriction:

- Subclasses of Autodesk.Revit.DB.Material- Subclasses of Autodesk.Revit.DB.CurveElement- Subclasses of Autodesk.Revit.DB.ConnectorElement- Subclasses of Autodesk.Revit.DB.HostedSweep- Autodesk.Revit.DB.Architecture.Room- Autodesk.Revit.DB.Mechanical.Space- Autodesk.Revit.DB.Area- Autodesk.Revit.DB.Architecture.RoomTag- Autodesk.Revit.DB.Mechanical.SpaceTag- Autodesk.Revit.DB.AreaTag- Autodesk.Revit.DB.CombinableElement- Autodesk.Revit.DB.Mullion- Autodesk.Revit.DB.Panel- Autodesk.Revit.DB.AnnotationSymbol- Autodesk.Revit.DB.Structure.AreaReinforcementType- Autodesk.Revit.DB.Structure.PathReinforcementType- Autodesk.Revit.DB.AnnotationSymbolType- Autodesk.Revit.DB.Architecture.RoomTagType- Autodesk.Revit.DB.Mechanical.SpaceTagType- Autodesk.Revit.DB.AreaTagType- Autodesk.Revit.DB.Structure.TrussType

The sample code that you provided looks perfectly correct to me, though. I do not see why it should not return any annotation symbols if there are any present in the model.

One idea for retrieving annotation symbols is to debug what goes on in RevitLookup. I created a new project and looked at the database contents using Add-Ins > Revit Lookup > Snoop DB... In the empty project, there was no entry for the AnnotationSymbol class.

I added an annotation symbol to the model using Annotate > Symbol and looked at the updated database contents in RevitLookup again, at which point the annotation symbol appears as expected:

![Snoop annotation symbol](img/snoop_annotation_symbol.png)