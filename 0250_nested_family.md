---
post_number: "0250"
title: "Nested Family"
slug: "nested_family"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'geometry', 'revit-api']
source_file: "0250_nested_family.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0250_nested_family.html"
---

### Nested Family

Here is another interesting question that came up on the creation of a nested family.

**Question:** Can you help me to create a nested family via the API?
I have a column family and a shelf family stored in RFA files, and I would like to implement a command which creates a new family with column and the shelf, and load it into a project.
How can this be achieved?

**Answer:** The solution to this task is quite easy.
I assume you have two existing families named column and shelf, and wish to create a third family hosting instances of these two.
The two families can be defined either in external RFA files or as in-memory documents.

Inserting instances of each of these into a third new family document means that we are working in the family context instead of the project context.
However, the steps to load and insert the family instances are almost identical in both environments:

- Load the family into the target document.- Select the symbol to insert.- Create a new family instance.

Here is a helper method InsertFamilySymbolFromRfa which implements these three steps:
```csharp
StructuralType \_non\_rst = StructuralType.NonStructural;

FamilyInstance InsertFamilySymbolFromRfa(
  string filename,
  Document doc )
{
  FamilyInstance fi = null;
  //
  // load family file:
  //
  Family f;
  if( doc.LoadFamily( filename, out f ) )
  {
    //
    // retrieve family symbol:
    //
    FamilySymbol symbol = null;
    foreach( FamilySymbol s in f.Symbols )
    {
      symbol = s;
      break;
    }
    //
    // create family instance:
    //
    if( null != symbol )
    {
      if( doc.IsFamilyDocument )
      {
        fi = doc.FamilyCreate.NewFamilyInstance(
          XYZ.Zero, symbol, \_non\_rst );
      }
      else
      {
        fi = doc.Create.NewFamilyInstance(
          XYZ.Zero, symbol, \_non\_rst );
      }
    }
  }
  return fi;
}
```

It accesses the NewFamilyInstance method on either the creation document returned by doc.Create in the project context, or on the family item factory returned by doc.FamilyCreate in the family one.

The NewFamilyInstance method provides several different overloads, and which one to use depends on the details of the families you are working with. In this case, we simply used the one that was easiest to call in this context.

Here is an example of a simple external command Execute method mainline with no error checking whatsoever calling this method to insert the two column and shelf families you mention:
```csharp
public IExternalCommand.Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  FamilyInstance a = InsertFamilySymbolFromRfa(
    "C:/tmp/column.rfa", doc );

  FamilyInstance b = InsertFamilySymbolFromRfa(
    "C:/tmp/shelf.rfa", doc );

  return IExternalCommand.Result.Succeeded;
}
```

I created both column.rfa and shelf.rfa from the metric column template with rather arbitrarily defined geometry for the column and shelf symbols. Here is the result of running the command in a new third target family document, which is also based on the metric column template:
![Nested column and shelf family instances](img/column_and_shelf.png)

Here is an
[updated version](zip/rfa_labs_20091028.zip)
of the
[Family API Labs](http://thebuildingcoder.typepad.com/blog/2009/10/revit-family-creation-api-labs.html)
that I will be using for my Family API virtual session at
[Autodesk University 2009](http://thebuildingcoder.typepad.com/blog/2009/10/au-2009.html)
including this new command as Lab 5, though only in the C# project.