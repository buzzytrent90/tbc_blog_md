---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: qa
optimization_date: '2025-12-11T11:44:14.013006'
original_url: https://thebuildingcoder.typepad.com/blog/0476_purge_text_note_types.html
post_number: '0476'
reading_time_minutes: 6
series: elements
slug: purge_text_note_types
source_file: 0476_purge_text_note_types.htm
tags:
- csharp
- elements
- filtering
- revit-api
- transactions
title: Purge Unused Text Note Types
word_count: 1123
---

### Purge Unused Text Note Types

Chuck Dodson of
[Scott&Goble Architects](http://www.scottgoble.com) and
my colleague Katsuaki Takamizawa came up with a question on how to purge unused elements from the Revit database, and Harry Mattison from the Revit API development team provides a partial answer:

**Question:** I would like to purge unused elements from the Revit project, such as line types, materials, fonts, etc., especially those imported from AutoCAD.
The AutoCAD API provides an IsUsed property for layers and other elements.
I have not found any Revit API method for purging.

Is there any way to determine if an element is not being used by any other elements and can be deleted safely?

**Answer:** The Revit API does not provide a generic member such as IsUsed.
Each type of element will have to be handled individually.

Here is one way to do it for text note types:
```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  ICollection<ElementId> textNoteTypes
    = collector.OfClass( typeof( TextNoteType ) )
      .ToElementIds()
      .ToList();

  FilteredElementCollector collectorUsed
    = new FilteredElementCollector( doc );

  ICollection<ElementId> textNotes
    = collectorUsed.OfClass( typeof( TextNote ) )
      .ToElementIds();

  foreach( ElementId textNoteid in textNotes )
  {
    TextNote textNote = doc.get\_Element(
      textNoteid ) as TextNote;

    bool removed = textNoteTypes.Remove(
      textNote.TextNoteType.Id );
  }

  doc.Delete( textNoteTypes );
```

Here is another approach:
```csharp
  ICollection<ElementId> textNotesTypeIds
    = new Collection<ElementId>();

  FilteredElementCollector textNoteCollector
    = new FilteredElementCollector( doc );

  ICollection<ElementId> textNotes
    = textNoteCollector.OfClass( typeof( TextNote ) )
      .ToElementIds();

  foreach( ElementId textNoteid in textNotes )
  {
    TextNote tn = doc.get\_Element(
      textNoteid ) as TextNote;

    textNotesTypeIds.Add( tn.TextNoteType.Id );
  }

  FilteredElementCollector unusedTypeCollector
    = new FilteredElementCollector( doc );

  ICollection<ElementId> unusedTypes
    = unusedTypeCollector
      .OfClass( typeof( TextNoteType ) )
      .Excluding( textNotesTypeIds )
      .ToElementIds();

  doc.Delete( unusedTypes );
```

Many thanks to Katsu-san and Harry for this hint!

Note the difference between the two:

- In the first approach, a collection A of all text note types is created, then all text notes are collected and iterated, removing each one's text note type from A.- In the second approach, a collection B is created of the text note types in use. Then the filtered element collector Excluding method is used to retrieve all text note types except those contained in B.

In both approaches, some simplification and optimisation of the text note iteration to determine the types in use can be achieved as follows, iterating over the collector directly instead of asking it for its element ids and then opening each text note element from its id:
```csharp
  FilteredElementCollector textNotes
    = new FilteredElementCollector( doc )
      .OfClass( typeof( TextNote ) );

  foreach( TextNote textNote in textNotes )
  {
    bool removed = textNoteTypes.Remove(
      textNote.TextNoteType.Id );
  }
```

I was curious whether the two approaches show any discernible performance difference, so I took this code and implemented a new Building Coder sample command CmdPurgeTextNoteTypes to benchmark the two.

Since the deletion operation is identical in both cases, I implemented two methods which just collect the unused text note types, GetUnusedTextNoteTypes and GetUnusedTextNoteTypesExcluding, and then common code can be used to delete them.

GetUnusedTextNoteTypes returns all unused text note types by collecting all existing types in the document and removing the ones in use afterwards:
```csharp
ICollection<ElementId> GetUnusedTextNoteTypes(
  Document doc )
{
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  ICollection<ElementId> textNoteTypes
    = collector.OfClass( typeof( TextNoteType ) )
      .ToElementIds()
      .ToList();

  FilteredElementCollector textNotes
    = new FilteredElementCollector( doc )
      .OfClass( typeof( TextNote ) );

  foreach( TextNote textNote in textNotes )
  {
    bool removed = textNoteTypes.Remove(
      textNote.TextNoteType.Id );
  }
  return textNoteTypes;
}
```

GetUnusedTextNoteTypesExcluding return all unused text note types by first determining all text note types in use and then collecting all the others using an exclusion filter:
```csharp
ICollection<ElementId>
  GetUnusedTextNoteTypesExcluding(
    Document doc )
{
  ICollection<ElementId> usedTextNotesTypeIds
    = new Collection<ElementId>();

  FilteredElementCollector textNotes
    = new FilteredElementCollector( doc )
      .OfClass( typeof( TextNote ) );

  foreach( TextNote textNote in textNotes )
  {
    usedTextNotesTypeIds.Add(
      textNote.TextNoteType.Id );
  }

  FilteredElementCollector unusedTypeCollector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( TextNoteType ) );

  if( 0 < usedTextNotesTypeIds.Count )
  {
    unusedTypeCollector.Excluding(
      usedTextNotesTypeIds );
  }

  ICollection<ElementId> unusedTypes
    = unusedTypeCollector.ToElementIds();

  return unusedTypes;
}
```

Note that we have to check whether usedTextNotesTypeIds is empty before calling the Excluding method.
If Excluding is called with an empty collection, it throws an exception.

Here is the code of the mainline benchmarking these two methods and deleting the unused text note types, using manual regeneration option and transaction mode:
```csharp
  UIApplication app = commandData.Application;
  Document doc = app.ActiveUIDocument.Document;

  ICollection<ElementId> unusedTextNoteTypes
    = GetUnusedTextNoteTypes( doc );

  int n = unusedTextNoteTypes.Count;

  int nLoop = 100;

  Stopwatch sw = new Stopwatch();

  sw.Reset();
  sw.Start();

  for( int i = 0; i < nLoop; ++ i )
  {
    unusedTextNoteTypes
      = GetUnusedTextNoteTypes( doc );

    Debug.Assert( unusedTextNoteTypes.Count == n,
      "expected same number of unused texct note types" );
  }

  sw.Stop();
  double ms = (double) sw.ElapsedMilliseconds
    / (double) nLoop;

  sw.Reset();
  sw.Start();

  for( int i = 0; i < nLoop; ++ i )
  {
    unusedTextNoteTypes
      = GetUnusedTextNoteTypesExcluding( doc );

    Debug.Assert( unusedTextNoteTypes.Count == n,
      "expected same number of unused texct note types" );
  }

  sw.Stop();
  double msExcluding
    = (double) sw.ElapsedMilliseconds
      / (double) nLoop;

  Transaction t = new Transaction( doc,
    "Purging unused text note types" );

  t.Start();

  sw.Reset();
  sw.Start();

  doc.Delete( unusedTextNoteTypes );

  sw.Stop();
  double msDeleting
    = (double) sw.ElapsedMilliseconds
      / (double) nLoop;

  t.Commit();

  Util.InfoMsg( string.Format(
    "{0} text note type{1} purged. "
    + "{2} ms to collect, {3} ms to collect "
    + "excluding, {4} ms to delete.",
    n, Util.PluralSuffix( n ),
    ms, msExcluding, msDeleting ) );

  return Result.Succeeded;
```

The assertions ensure that the number of unused text note types is always the same.
I was worried whether the assertions would affect the benchmarking results, so I ran two sets of tests, one with the assertions commented out, but that makes no difference.

Here are some results in various sample models, with and without assertions, listing the

- M – Sample model:
  - A – a small model- B – rac\_basic\_sample\_project.rvt- C – rac\_advanced\_sample\_project.rvt- D – a slightly larger model- N – number of unused text note types purged- C – milliseconds required to collect- E – milliseconds required to collect excluding- D – milliseconds required to delete

The first row of each pair shows the time with assertions, the second without:

| M | N | C | E | D |
| --- | --- | --- | --- | --- |
| A | 7 | 4.97 | 4.35 | 0.07 |
| A | 7 | 4.43 | 4.39 | 0.07 |
| B | 1 | 5.8 | 5.25 | 0.12 |
| B | 1 | 6.4 | 5.38 | 0.29 |
| C | 3 | 23.45 | 29.7 | 0.03 |
| C | 3 | 24.29 | 28.41 | 0.03 |
| D | 0 | 89.91 | 107.73 | 0.00 |
| D | 0 | 89.18 | 108.94 | 0.00 |

As you can see, the performance of the two methods is similar, with an advantage for the Excluding method in small models and a disadvantage in larger ones.
If you seriously want to optimise a similar algorithm, you will have to decide based on the characteristics of the models you typically work with.
I am pretty certain that the total number of text notes and used versus unused text note types will play a significant role.

Here is
[version 2011.0.79.0](zip/bc_11_79.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.