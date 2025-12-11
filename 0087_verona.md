---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: documentation
optimization_date: '2025-12-11T11:44:13.338675'
original_url: https://thebuildingcoder.typepad.com/blog/0087_verona.html
post_number: 0087
reading_time_minutes: 2
series: general
slug: verona
source_file: 0087_verona.htm
tags:
- csharp
- elements
- family
- filtering
- revit-api
- selection
- walls
title: Verona Revit API Training
word_count: 490
---

### Verona Revit API Training

Yesterday was the last day of the advanced Revit Structure API training at
[Steel & Graphics srl](http://www.steel-graphics.com)
in Verona, Italy, with Gianni, Stefano, Michele, Paolo, and we started diving into deeper topics.

![Gianni, Michele, Jeremy and Stefano](img/verona_2_380.jpg)

But first, here are some of the initial things we discussed in the previous days:

- Making use of FamilyInstance.AddCoping to implement a Boolean operation between solids, for instance to cut a hole in a steel beam.
- Computing the volumes of all compound wall layers.
- Creating new column and beam types and inserting column and beam instances through the API.
- Improving the Revit SDK FrameBuilder sample to use language independent category comparison and make use of the Revit 2009 API filtering mechanism.

I already posted a description of the changes made to FrameBuilder to make the
[category comparison](http://thebuildingcoder.typepad.com/blog/2009/01/category-comparison.html)
language independent.
Notes on adding filtering may follow later.
Here are some notes on the first of these issues:

#### AddCoping

Making use of AddCoping is pretty trivial.
All we need to do is select two suitable beams, for instance a wide steel flange beam to cut a hole into, and a steel bar to define the hole, then feed these two instances into the AddCoping method.
First of all, we implemented the following method to either extract a preselected element from the document selection set, or prompt the user to select a new one:

```csharp
Element GetSelectedElement(
  Document doc,
  string description )
{
  Selection sel = doc.Selection;
  Element e = null;
  while( null == e )
  {
    ElementSetIterator i
      = sel.Elements.ForwardIterator();

    if( i.MoveNext() )
    {
      e = i.Current as Element;
      sel.Elements.Remove( e );
    }
    if( null == e )
    {
      sel.Elements.Clear();

      sel.StatusbarTip = "Please select "
        + description;

      if( !sel.PickOne() )
      {
        return null;
      }
    }
  }
  return e;
}
```

Then we can make use of that to select the two elements in the correct order and call the AddCoping method on the larger beam:

```csharp
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  string s
    = "Please select a family instance for the ";

  IExternalCommand.Result rc
    = IExternalCommand.Result.Failed;

  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  Element e1 = GetSelectedElement( doc,
    "beam to cut" );

  if( null != e1 )
  {
    Element e2 = GetSelectedElement( doc,
      "bar to drill hole" );

    if( null != e2 )
    {
      FamilyInstance beam = e1 as FamilyInstance;
      FamilyInstance bar = e2 as FamilyInstance;

      if( null == beam )
      {
        message = s + "beam";
      }
      else if( null == bar )
      {
        message = s + "bar";
      }
      else
      {
        beam.AddCoping( bar );
        rc = IExternalCommand.Result.Succeeded;
      }
    }
  }
  return rc;
}
```

The result of executing this is the flange beam with a nice little hole in it:

![Beam with a hole cut by AddCoping](img/AddCoping.png)

Unfortunately, this cannot be used as a generic Boolean operation, because if the bar is erased from the project, the hole immediately and automatically seals up again.

I hope to be able to jot down some notes on the other topics listed above on the way back home.