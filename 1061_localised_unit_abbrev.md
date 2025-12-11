---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.3
content_type: code_example
optimization_date: '2025-12-11T11:44:15.218950'
original_url: https://thebuildingcoder.typepad.com/blog/1061_localised_unit_abbrev.html
post_number: '1061'
reading_time_minutes: 5
series: general
slug: localised_unit_abbrev
source_file: 1061_localised_unit_abbrev.htm
tags:
- csharp
- elements
- parameters
- python
- revit-api
- sheets
- views
- windows
title: Localised Unit Abbreviations
word_count: 1028
---

### Localised Unit Abbreviations

I recently discussed how to generate
[unit abbreviations](http://thebuildingcoder.typepad.com/blog/2013/11/unit-abbreviations.html),
both using a
[hard coded list](http://thebuildingcoder.typepad.com/blog/2013/11/unit-abbreviations.html#2) and
automatically generating them
[from the UnitSymbolType enumeration values](http://thebuildingcoder.typepad.com/blog/2013/11/unit-abbreviations.html#3).

I implemented the CmdDutAbbreviation command in
[The Building Coder samples](http://thebuildingcoder.typepad.com/blog/2013/10/the-building-coder-samples-on-github.html) to
test these abbreviations, and also enhanced it to display the
[Display Unit Type to Unit Types mapping](http://thebuildingcoder.typepad.com/blog/2013/11/mapping-display-unit-type-to-unit-types.html) that
we added in the meantime.

Victor Chekalin, or Виктор Чекалин, reacted to this analysis and adds:

> Thank you for your investigation.
>
> But... There is a big 'but' :-)
>
> Your method with UnitSymbolType is not completely suitable for localised applications, e.g. Russian.
>
> As always, I have solved these little issues :-)
>
> You write above: 'After some further digging on the Revit API help file RevitAPI.chm, I discovered the FormatOptions GetValidUnitSymbols method'. But you did not dig deep enough to find the old GetLabelFor method in the static LabelUtils class.
>
> If you looked at that, you'll see that one of overridden method takes UnitSymbolType as parameter.
>
> And guess what it returns?
>
> Right. The **localizable** abbreviation for Unit Symbol :-) – exactly what we need.
>
> So, the command to retrieve a suitable localised Unit Type abbreviation becomes much simpler – check out
> <http://pastebin.com/ZZ0mVeYE> –
> regards, Victor:

```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  DisplayUnitType n = DisplayUnitType.DUT\_GALLONS\_US;

  for( DisplayUnitType i = DisplayUnitType
    .DUT\_METERS; i < n; ++i )
  {
    var validUnitSymbols = FormatOptions
      .GetValidUnitSymbols( i );

    foreach( var validUnitSymbol in validUnitSymbols )
    {
      if( validUnitSymbol != UnitSymbolType.UST\_NONE )
      {
        var abbrUnitTypeLabel = LabelUtils.GetLabelFor(
          validUnitSymbol );

        Debug.Print( "{0} - {1}", abbrUnitTypeLabel,
          LabelUtils.GetLabelFor( i ) );
      }
    }
  }
  return Result.Succeeded;
}
```

Jeremy adds: I updated
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) and
integrated Victor's enhancement to the existing CmdDutAbbreviation command.

The relevant code looks like this now and lists both the official Revit API LabelUtils and hard-coded abbreviations plus the valid unit symbols for the first 26 display unit types:

```csharp
  MapDutToUt map\_dut\_to\_ut = new MapDutToUt();

  DisplayUnitType n
    = DisplayUnitType.DUT\_GALLONS\_US;

  Debug.Print( "Here is a list of the first {0} "
    + "display unit types with official Revit API "
    + "LabelUtils, hard-coded The Building Coder "
    + "abbreviations and valid unit symbols:\n",
    (int) n - 1 );

  string unit\_types, valid\_unit\_symbols;

  for( DisplayUnitType i = DisplayUnitType
    .DUT\_METERS; i < n; ++i )
  {
    List<string> uts = new List<string>(
      map\_dut\_to\_ut[i]
        .Select<UnitType, string>(
          u => u.ToString().Substring( 3 ) ) );

    int m = uts.Count;

    unit\_types = 4 > m
      ? string.Join( ", ", uts )
      : string.Format( "{0}, {1} and {2} more",
        uts[0], uts[1], m - 2 );

    valid\_unit\_symbols = string.Join( ", ",
      FormatOptions.GetValidUnitSymbols( i )
        .Where( u => UnitSymbolType.UST\_NONE != u )
        .Select<UnitSymbolType, string>(
          u => LabelUtils.GetLabelFor( u )
            + "/" + Util.UnitSymbolTypeString( u ) ) );

    Debug.Print( "{0,6} - {1} - {2}: {3}",
      Util.DisplayUnitTypeAbbreviation[(int) i],
      LabelUtils.GetLabelFor( i ),
      unit\_types,
      valid\_unit\_symbols );
  }
```

On my system, this generates the following output in the Visual Studio debug output window (copy and paste somewhere or view source to see truncated lines in full):

```
Here is a list of the first 26 display unit types with
official Revit API LabelUtils, hard-coded The Building
Coder abbreviations and valid unit symbols:

    m - Meters - Length, SheetLength and 19 more: m/m
   cm - Centimeters - Length, SheetLength and 20 more: cm/cm
   mm - Millimeters - Length, SheetLength and 20 more: mm/mm
   ft - Decimal feet - Length, SheetLength and 19 more: '/foot_single_quote, LF/lf
  N/A - Feet and fractional inches - Length, SheetLength and 19 more:
  N/A - Fractional inches - Length, SheetLength and 20 more:
   in - Decimal inches - Length, SheetLength and 20 more: "/inch_double_quote
   ac - Acres - Area, HVAC_CrossSection: acres/acres
   ha - Hectares - Area, HVAC_CrossSection: hectare/hectares
  N/A - Meters and centimeters - Length, SheetLength and 17 more:
  y^3 - Cubic yards - Volume, Piping_Volume: CY/cy
 ft^2 - Square feet - Area, HVAC_CrossSection and 2 more: SF/sf, ft²/ft^2
  m^2 - Square meters - Area, HVAC_CrossSection and 2 more: m²/m^2
 ft^3 - Cubic feet - Volume, Piping_Volume and 2 more: CF/cf, ft³/ft^3
  m^3 - Cubic meters - Volume, Piping_Volume and 2 more: m³/m^3
  deg - Decimal degrees - Angle, SiteAngle, Rotation: °/degree_symbol
  N/A - Degrees minutes seconds - Angle, SiteAngle, Rotation:
  N/A - General - Number:
  N/A - Fixed - Number, HVAC_Factor, Electrical_Demand_Factor:
    % - Percentage - Number, Slope and 4 more: %/percent_sign
 in^2 - Square inches - Area, HVAC_CrossSection and 2 more: in²/in^2
 cm^2 - Square centimeters - Area, HVAC_CrossSection and 2 more: cm²/cm^2
 mm^2 - Square millimeters - Area, HVAC_CrossSection and 2 more: mm²/mm^2
 in^3 - Cubic inches - Volume, Piping_Volume and 2 more: in³/in^3
 cm^3 - Cubic centimeters - Volume, Piping_Volume and 2 more: cm³/cm^3
 mm^3 - Cubic millimeters - Volume, Piping_Volume, Section_Modulus: mm³/mm^3
    l - Liters - Volume, Piping_Volume: L/l
```

Quite a nice correspondence, I think.

Note that the official Revit API LabelUtils abbreviations are localised, as Victor underlines, whereas the hard-coded ones obviosuly are not.

This discussion once again highlights several aspects:

- The Revit API provides a huge amount of functionality, sometimes more than you expect.
- Some functionality is not always obvious to find.
- The various \*Utils classes contain many of these hidden gems, as Rudi likes to point out.

The updated version described above is published as
[release 2014.0.105.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.105.2) and
includes the release 2014.0.105.1
[Display Unit Type to Unit Types mapping](http://thebuildingcoder.typepad.com/blog/2013/11/mapping-display-unit-type-to-unit-types.html).

#### Full Moon Celebration

I often take the opportunity of spending a couple of hours enjoying infinite space, time, air, fire, earth, and sometimes water by celebrating the full moon with a little fire on a hill nearby with a good view all around.

I did so last night as well, and it was very nice.

Here is a farewell picture of the full moon early this morning:

![Morning full moon](img/morning_full_moon_2013-11.jpeg)