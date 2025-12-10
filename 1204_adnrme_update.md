---
post_number: "1204"
title: "AdnRme Update to Eliminate Obsolete API Usage"
slug: "adnrme_update"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'parameters', 'revit-api', 'views']
source_file: "1204_adnrme_update.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1204_adnrme_update.html"
---

### AdnRme Update to Eliminate Obsolete API Usage

The ADN Revit MEP HVAC and electrical sample
[AdnRme](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.40) version
[2015.0.0.2](https://github.com/jeremytammik/AdnRme/releases/tag/2015.0.0.2) produces
[three compilation warnings](zip/AdnRme_2015_2_warnings.txt),
all three saying:

- 'Autodesk.Revit.DB.Family.Symbols' is obsolete:
  'This property is obsolete in Revit 2015.
  Use Family.GetFamilySymbolIds() instead.'

So let's do what the man says.

The code producing the first two instances of the warning in the module CmdChangeSize.cs looks like this:

```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( Family ) );

  foreach( Family family in collector )
  {
    // Family category is not implemented,
    // so check the symbols instead:

    bool categoryMatches = false;
    foreach( FamilySymbol symbol in family.Symbols )
    {
      categoryMatches = ( null != symbol.Category
        && symbol.Category.Id.IntegerValue.Equals(
          ( int ) BuiltInCategory.OST\_DuctTerminal ) );

      break; // we only need to check the first one
    }
    if( categoryMatches )
    {
      List<SymbMinMax> familySymbols
        = new List<SymbMinMax>();

      foreach( FamilySymbol symbol in family.Symbols )
      {
        SymbMinMax a = new SymbMinMax();
        a.Symbol = symbol;

        a.Min = Util.GetParameterValueFromName(
          symbol, ParameterName.MinFlow );

        a.Max = Util.GetParameterValueFromName(
          symbol, ParameterName.MaxFlow );

        familySymbols.Add( a );
      }
      dictFamilyToSymbols.Add(
        family.Name, familySymbols );
    }
  }
```

This is the replacement code:

```csharp
  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( Family ) );

  foreach( Family family in collector )
  {
    ISet<ElementId> symbolIds
      = family.GetFamilySymbolIds();

    // Family category is not implemented,
    // so check the symbols instead:

    bool categoryMatches = false;

    foreach( ElementId id in symbolIds )
    {
      Element symbol = doc.GetElement( id );

      categoryMatches = ( null != symbol.Category
        && symbol.Category.Id.IntegerValue.Equals(
          (int) BuiltInCategory.OST\_DuctTerminal ) );

      break; // we only need to check the first one
    }
    if( categoryMatches )
    {
      List<SymbMinMax> familySymbols
        = new List<SymbMinMax>();

      foreach( ElementId id in symbolIds )
      {
        FamilySymbol symbol = doc.GetElement( id )
          as FamilySymbol;

        SymbMinMax a = new SymbMinMax();
        a.Symbol = symbol;

        a.Min = Util.GetParameterValueFromName(
          symbol, ParameterName.MinFlow );

        a.Max = Util.GetParameterValueFromName(
          symbol, ParameterName.MaxFlow );

        familySymbols.Add( a );
      }
      dictFamilyToSymbols.Add(
        family.Name, familySymbols );
    }
  }
```

Note that in the first replacement above, we skip casting the element returned by the GetElement method to a FamilySymbol instance, because all we need to do with it is query its Category property.
This can be done just as well on the Element base class.

The second replacement requires the cast, however.

You can view the GitHub
[diff between version 2015.0.0.2 and 2015.0.0.3](https://github.com/jeremytammik/AdnRme/compare/2015.0.0.2...2015.0.0.3) to
see the exact changes I made to eliminate all three warnings.

I added some other trivial changes after the fix described above, in fact, so the current version at the moment of writing this is
[2015.0.0.4](https://github.com/jeremytammik/AdnRme/releases/tag/2015.0.0.4).

You can always download the complete source code, Visual Studio solution and add-in manifest of the most up-to-date version from the
[AdnRme GitHub repository](https://github.com/jeremytammik/AdnRme).