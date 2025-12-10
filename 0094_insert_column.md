---
post_number: "0094"
title: "Inserting a Column"
slug: "insert_column"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'references', 'revit-api']
source_file: "0094_insert_column.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0094_insert_column.html"
---

### Inserting a Column

Here I continue the discussion of another of the topics that were raised in the
[Revit API training in Verona](http://thebuildingcoder.typepad.com/blog/2009/01/verona-revit-api-training.html)
last week, on creating new column types and inserting column instances through the API.

We explored the following topics:

- Retrieving all family elements in current document in order to check whether the family we are interested in is loaded.
- Exploring the results of using a FamilyFilter.
- Loading a new family, if not already present.
- Creating a new family symbol, i.e. duplicating a type and setting its name and dimensions.
- Inserting a new column instance into the model.

We implemented a new external command CmdNewColumnTypeInstance to demonstrate these steps. First, we set up some constants to define the family we are interested in, our library path, the structural type and a unit conversion:

```csharp
const string family\_name
  = "M\_Rectangular Column";

const string extension
  = ".rfa";

const string directory
  = "C:/Documents and Settings/All Users"
  + "/Application Data/Autodesk/RAC 2009"
  + "/Metric Library/Columns/";

const string path
  = directory + family\_name + extension;

StructuralType nonStructural
  = StructuralType.NonStructural;

const double \_foot\_to\_mm
  = 25.4 \* 12;
```

Here is the implementation of the Execute method which achieves the steps listed above:

In order to check whether the family we are interested in is already loaded, we make use of a family filter to get all family elements in the current document.
If so, we store it in the variable 'f'.

The family filter returns both the symbols contained within the family and the family itself.
In real life, we would probably eliminate the symbols by creating a Boolean 'and' filter and filtering for the Family class as well as the family name.

```csharp
CmdResult rc
  = CmdResult.Failed;

Application app = commandData.Application;
Document doc = app.ActiveDocument;
List<Element> symbols = new List<Element>();

Filter filter = app.Create.Filter.NewFamilyFilter(
  family\_name );

doc.get\_Elements( filter, symbols );
Family f = null;
foreach( Element e in symbols )
{
  if( e is Family )
  {
    f = e as Family;
  }
  else if( e is FamilySymbol )
  {
    FamilySymbol s = e as FamilySymbol;
    Debug.Print(
      "Family name={0}, symbol name={1}",
      s.Family.Name, s.Name );
  }
}
```

If the family was not already loaded, then 'f' remains null, and we load it with the LoadFamily method.
It would also be sufficient to load one single symbol from the family:

```csharp
if( null == f )
{
  if( !doc.LoadFamily( path, ref f ) )
  {
    message = "Unable to load '" + path + "'.";
  }
}
```

If the loading does not succeed, 'f' is still null and we cannot proceed.
Otherwise, we pick one of them for duplication and store it in the variable 's'.
Any one will do, so we simply select the first one.

```csharp
if( null != f )
{
  Debug.Print( "Family name={0}", f.Name );
  FamilySymbol s = null;
  foreach( FamilySymbol s2 in f.Symbols )
  {
    s = s2;
    break;
  }
  Debug.Assert( null != s,
    "expected at least one symbol"
    + " to be defined in family" );
```

We duplicate it using the Duplicate method, simultaneously defining the new symbol name.
To find out what parameters are available for setting the new type's dimensions, we first list all of them.
We set the new type's dimensions and demonstrate that we can change its name at a later stage as well if desired.
By trial and error, we definitely proved that when using a localised name to set the parameter value, the specified name is case sensitive.

```csharp
  Symbol s1 = s.Duplicate( "Nuovo simbolo" );
  s = s1 as FamilySymbol;
  foreach( Parameter param in s.Parameters )
  {
    Debug.Print(
      "Parameter name={0}, value={1}",
      param.Definition.Name,
      param.AsValueString() );
  }
  s.get\_Parameter( "Width" ).Set(
    500 / \_foot\_to\_mm );

  s.get\_Parameter( "Depth" ).Set(
    1000 / \_foot\_to\_mm );
  s.Name = "Nuovo simbolo due";
```

Finally, we insert an instance of our new symbol. We also tested inserting it using a non-vertical reference direction, but that is ignored for columns, because columns are inherently vertical:

```csharp
  XYZ p = XYZ.Zero;
  doc.Create.NewFamilyInstance(
    p, s, nonStructural );
  rc = CmdResult.Succeeded;
}
return rc;
```

The next step we took was to explore the similar concepts for beams and braces, but the discussion of those steps will have to wait until some later time.

Here is
[version 1.0.0.23](http://thebuildingcoder.typepad.com/blog/files/bc10023.zip)
of the complete Visual Studio solution with this new command implementation.