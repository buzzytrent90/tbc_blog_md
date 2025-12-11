---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.1
content_type: code_example
optimization_date: '2025-12-11T11:44:13.353494'
original_url: https://thebuildingcoder.typepad.com/blog/0097_list_railing_types.html
post_number: 0097
reading_time_minutes: 3
series: elements
slug: list_railing_types
source_file: 0097_list_railing_types.htm
tags:
- csharp
- elements
- family
- filtering
- python
- revit-api
title: List Railing Types
word_count: 606
---

### List Railing Types

This is in response to a query from Berria on
[iterating over the railing types](http://thebuildingcoder.typepad.com/blog/2009/02/inserting-a-column.html#comments)
in the model.
It is also once again about teaching how to fish, and not just feeding people, at least I hope so.
If I want to list all the railing types, the best way to go is probably to define an API filter which retrieves them from the model for me.
How do I design that filter?
I start by opening a new model, inserting a railing, and examining it with
[RvtMgdDbg](http://download.autodesk.com/media/adn/RvtMgdDbg2009_0429_2008.zip).
There I can easily determine that its built-in category is OST\_StairsRailing.
Therefore, I first thought that an API filter selecting all family symbols of this category should suffice.
The project browser shows me the following stair and railing symbols in my current model:

![Railing and stair types in project browser](img/browser_railings_and_stairs.png)

I implemented a new external command CmdListRailingTypes to test this approach, with the following code:

```python
Application app = commandData.Application;
Document doc = app.ActiveDocument;
CreationFilter cf = app.Create.Filter;

List<Element> symbols = new List<Element>();

BuiltInCategory bic
  = BuiltInCategory.OST\_StairsRailing;

Filter f1
  = cf.NewCategoryFilter( bic );

Filter f2
  = cf.NewTypeFilter( typeof( FamilySymbol ) );

doc.get\_Elements( f, symbols );
foreach( FamilySymbol s in symbols )
{
  Debug.Print(
    "Family name={0}, symbol name={1}",
    s.Family.Name, s.Name );
}
return CmdResult.Failed;
```

It makes use of a new alias for CreationFilter:

```csharp
using CreationFilter
  = Autodesk.Revit.Creation.Filter;
```

To my initial surprise, no symbols were found. I thereupon took a closer look at the symbols within the railings family in RvtMgdDbg and discovered that the category of the M\_Baluster symbols is not OST\_StairsRailing but OST\_StairsRailingBaluster. Changing the value of the 'bic' variable to that built-in category returns some valid results:

```
Family name=M_Baluster - Square, symbol name=25mm
Family name=M_Baluster - Square, symbol name=20mm
Family name=M_Baluster - Round, symbol name=25mm
Family name=M_Baluster - Round, symbol name=20mm
```

This does still not include all the railing symbols listed in the browser, so I went back into RvtMgdDbg and searched for those as well. They were not listed in the FamilySymbol collection, but under Symbols, this time indeed with the category OST\_StairsRailing.

Making use of this info, I implemented a second filter and iteration to retrieve and list Symbol objects like this:

```csharp
bic = BuiltInCategory.OST\_StairsRailing;
f1 = cf.NewCategoryFilter( bic );
f2 = cf.NewTypeFilter( typeof( Symbol ) );
f = cf.NewLogicAndFilter( f1, f2 );

doc.get\_Elements( f, symbols );

n = symbols.Count;

Debug.Print( "{0}"
  + " OST\_StairsRailing symbol{1}:",
  n, Util.PluralSuffix( n ) );

foreach( Symbol s in symbols )
{
  FamilySymbol fs = s as FamilySymbol;

  Debug.Print(
    "Family name={0}, symbol name={1}",
    null == fs ? "<none>" : fs.Family.Name,
    s.Name );
}
```

The result of this code is somewhat surprising too.
The non-family symbols that we are looking for are retrieved now, just like we hoped, and we also retrieve the family symbols that we already saw above:

```
6 OST_StairsRailing symbols:
Family name=M_Baluster - Square, symbol name=25mm
Family name=M_Baluster - Square, symbol name=20mm
Family name=M_Baluster - Round, symbol name=25mm
Family name=M_Baluster - Round, symbol name=20mm
Family name=<none>, symbol name=900mm Pipe
Family name=<none>, symbol name=900mm Rectangular
```

This goes to show several things:

- You need to explore the model.
- You may get unexpected results.

I'll just leave it at that for now.

Here is
[version 1.0.0.25](http://thebuildingcoder.typepad.com/blog/files/bc10025.zip)
of the complete Visual Studio solution with the new CmdListRailingTypes command implementation.