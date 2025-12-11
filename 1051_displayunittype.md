---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.200059'
original_url: https://thebuildingcoder.typepad.com/blog/1051_displayunittype.html
post_number: '1051'
reading_time_minutes: 6
series: elements
slug: displayunittype
source_file: 1051_displayunittype.htm
tags:
- csharp
- revit-api
- sheets
- views
- elements
title: Mapping Display Unit Type to Unit Types
word_count: 1242
---

### Mapping Display Unit Type to Unit Types

Yesterday, we looked at
[unit abbreviations](http://thebuildingcoder.typepad.com/blog/2013/11/unit-abbreviations.html) and
some other aspects of the new Revit 2014
[Unit API](http://thebuildingcoder.typepad.com/blog/2013/11/unit-abbreviations.html#0).

I mentioned that one might be able to use the FormatUtils.Format or UnitFormatUtils.Format methods to generate a string representation of a given numeric value, extract a unit abbreviation from it, and that I was not looking further at that option, lacking a UnitType to feed it with, only having a DisplayUnitType to work with.

That led me to wonder whether there might be an easy way to find out what UnitType a given DisplayUnitType corresponds to.

I was naively expecting to be able to find a simple one-to-many correspondence, e.g. a UnitType for length might be mapped to the various display unit types such as mm, cm, m, in, ft, etc., so each display unit type would only be mapped to one single unit type.

As it turns out, this mapping is easy to discover, since a call to the UnitUtils.GetValidDisplayUnits method passing in a given UnitType will return a list of all DisplayUnitTypes that it might be used for.

On the other hand, the mapping is not a simple 1-to-N one, but N-to-N, because the unit types include both the simple basic types, such as length, and complex or specialised ones, such as SheetLength, HVAC\_DuctSize etc., so there are a number of different length related unit types, all of which can be expressed in any length unit.

For instance, looking more closely at the display unit type DUT\_METERS, there are no less than 21 different valid unit types in which it may appear:

- Length, SheetLength, HVAC\_DuctSize, HVAC\_Roughness, PipeSize, Piping\_Roughness, WireSize, DecSheetLength, Electrical\_CableTraySize, Electrical\_ConduitSize, Reinforcement\_Length, HVAC\_DuctInsulationThickness, HVAC\_DuctLiningThickness, PipeInsulationThickness, Bar\_Diameter, Crack\_Width, Displacement\_Deflection, Reinforcement\_Cover, Reinforcement\_Spacing, Section\_Dimension, Section\_Property.

I implemented a mapping dictionary MapDutToUt returning the valid unit types for any given display unit type and added a call to list its results to the
[CmdDutAbbreviation external command](http://thebuildingcoder.typepad.com/blog/2013/11/unit-abbreviations.html#4).

The mapping is initialised by calling the constructor, which is the only method required in addition to the inherited .NET generic dictionary functionality:

```csharp
/// <summary>
/// Map each DisplayUnitType to a list of all the
/// UnitType values that it might be used for, e.g.
/// Meters is mapped to the following 21 values:
/// Length, SheetLength, HVAC\_DuctSize, HVAC\_Roughness,
/// PipeSize, Piping\_Roughness, WireSize, DecSheetLength,
/// Electrical\_CableTraySize, Electrical\_ConduitSize,
/// Reinforcement\_Length, HVAC\_DuctInsulationThickness,
/// HVAC\_DuctLiningThickness, PipeInsulationThickness,
/// Bar\_Diameter, Crack\_Width, Displacement\_Deflection,
/// Reinforcement\_Cover, Reinforcement\_Spacing,
/// Section\_Dimension, Section\_Property.
/// </summary>
class MapDutToUt : Dictionary<DisplayUnitType, List<UnitType>>
{
  public MapDutToUt()
  {
    IList<DisplayUnitType> duts;

    Array a = Enum.GetValues( typeof( UnitType ) );

    foreach( UnitType ut in a )
    {
      // Skip the UT\_Undefined and UT\_Custom entries;
      // GetValidDisplayUnits throws ArgumentException
      // on them, saying "unitType is an invalid unit
      // type.  See UnitUtils.IsValidUnitType() and
      // UnitUtils.GetValidUnitTypes()."

      if( UnitType.UT\_Undefined == ut
        || UnitType.UT\_Custom == ut )
      {
        continue;
      }

      duts = UnitUtils.GetValidDisplayUnits( ut );

      foreach( DisplayUnitType dut in duts )
      {
        //Debug.Assert( !ContainsKey( dut ),
        //  "unexpected duplicate DisplayUnitType key" );

        if( !ContainsKey( dut ) )
        {
          Add( dut, new List<UnitType>( 1 ) );
        }
        this[dut].Add( ut );
      }
    }
  }
}
```

The UnitUtils.GetValidDisplayUnits method must not be called with the UT\_Undefined or UT\_Custom entries, or it will throw an ArgumentException saying "unitType is an invalid unit type. See UnitUtils.IsValidUnitType() and UnitUtils.GetValidUnitTypes()."

I added a call to instantiate this mapping and report its results to the external command like this:

```csharp
  MapDutToUt map\_dut\_to\_ut = new MapDutToUt();

  DisplayUnitType n
    = DisplayUnitType.DUT\_GALLONS\_US;

  Debug.Print( "Here is a list of the first {0} "
    + "display unit types with The Building Coder "
    + "abbreviation and the valid unit symbols:\n",
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
        .Select<UnitSymbolType, string>(
          u => Util.UnitSymbolTypeString( u ) ) );

    Debug.Print( "{0,6} - {1} - {2}: {3}",
      Util.DisplayUnitTypeAbbreviation[(int) i],
      LabelUtils.GetLabelFor( i ),
      unit\_types,
      valid\_unit\_symbols );
  }
```

Note that I did not want to see the entire long list of valid unit types for each display unit type, since some of them are rather lengthy.

Therefore, only the first two or three are displayed, and the number of additional ones mentioned in case there are more, like this (copy and paste somewhere or view source to see truncated lines in full):

```
    m - Meters - Length, SheetLength and 19 more: none, m
   cm - Centimeters - Length, SheetLength and 20 more: none, cm
   mm - Millimeters - Length, SheetLength and 20 more: none, mm
   ft - Decimal feet - Length, SheetLength and 19 more: none, foot_single_quote, lf
  N/A - Feet and fractional inches - Length, SheetLength and 19 more: none
  N/A - Fractional inches - Length, SheetLength and 20 more: none
   in - Decimal inches - Length, SheetLength and 20 more: none, inch_double_quote
   ac - Acres - Area, HVAC_CrossSection: none, acres
   ha - Hectares - Area, HVAC_CrossSection: none, hectares
  N/A - Meters and centimeters - Length, SheetLength and 17 more: none
  y^3 - Cubic yards - Volume, Piping_Volume: none, cy
 ft^2 - Square feet - Area, HVAC_CrossSection and 2 more: none, sf, ft^2
  m^2 - Square meters - Area, HVAC_CrossSection and 2 more: none, m^2
 ft^3 - Cubic feet - Volume, Piping_Volume and 2 more: none, cf, ft^3
  m^3 - Cubic meters - Volume, Piping_Volume and 2 more: none, m^3
  deg - Decimal degrees - Angle, SiteAngle, Rotation: none, degree_symbol
  N/A - Degrees minutes seconds - Angle, SiteAngle, Rotation: none
  N/A - General - Number: none
  N/A - Fixed - Number, HVAC_Factor, Electrical_Demand_Factor: none
    % - Percentage - Number, Slope and 4 more: none, percent_sign
 in^2 - Square inches - Area, HVAC_CrossSection and 2 more: none, in^2
 cm^2 - Square centimeters - Area, HVAC_CrossSection and 2 more: none, cm^2
 mm^2 - Square millimeters - Area, HVAC_CrossSection and 2 more: none, mm^2
 in^3 - Cubic inches - Volume, Piping_Volume and 2 more: none, in^3
 cm^3 - Cubic centimeters - Volume, Piping_Volume and 2 more: none, cm^3
 mm^3 - Cubic millimeters - Volume, Piping_Volume, Section_Modulus: none, mm^3
    l - Liters - Volume, Piping_Volume: none, l
```

The updated version of the command is available from
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples) and
the version discussed above is
[release 2014.0.105.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.105.1).

I found this little exploration very illuminating and hope that you learned something from it as well.

#### Why No Backward Compatibility

Developers occasionally ask why the Revit API does not go to similarly extreme lengths as the AutoCAD API to support backward compatibility.

The simple answer is that this enables to Revit API developers to improve the API better.

It costs a huge amount of additional effort to maintain backwards compatibility in an API.

The Revit API team tries to move forwards faster and not look back all the time.

This hardly ever led to any really significant upheaval, although it does mean that most new major releases require a recompilation.

Most
[intermediate API update releases do not](http://thebuildingcoder.typepad.com/blog/2011/11/intermediate-api-update-releases.html),
though.