---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 7.1
content_type: code_example
optimization_date: '2025-12-11T11:44:14.397881'
original_url: https://thebuildingcoder.typepad.com/blog/0688_unit_display_str.html
post_number: 0688
reading_time_minutes: 11
series: general
slug: unit_display_str
source_file: 0688_unit_display_str.htm
tags:
- csharp
- elements
- family
- geometry
- parameters
- python
- revit-api
- selection
- sheets
- transactions
- views
- walls
- windows
title: Unit Conversion and Display String Formatting
word_count: 2239
---

﻿

### Unit Conversion and Display String Formatting

AU went very well for me, and I think this was the one I liked most of all so far, to my own surprise.
Now I am already at the next conference in Moscow, from where I continue to Tel Aviv tomorrow.
December is always my monster travelling month, and I never get to prepare for Christmas or enjoy the dark and cosy celebration of
[Advent](http://en.wikipedia.org/wiki/Advent).
But I really did have fun and enjoy AU in Las Vegas.

Sunday morning my colleague Marat Mirgaleev invited me to join him in his weekly volleyball game, which was a wonderful break from the conference presentation preparation.
Marat is also a member of the ADN DevTech team and spelled Марат Миргалеев in Cyrillic.
Another Autodesk colleague who also joined was Rustam Ibragimov, Рустам Ибрагимов.
Here are Rustam, Marat, I and our all-time star, the volleyball herself:

![Rustam, Marat, Jeremy and the volleyball](img/rustam_marat_jeremy_volleyball.jpg)

Here we are now in the Autodesk Moscow office:

![DevDay conference in Moscow](img/devday_moscow_2011-12-05_8311.jpg)

Fittingly enough, here is a
[question](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html?cid=6a00e553e1689788330154379044c3970c#comment-6a00e553e1689788330154379044c3970c) from
Russia, by Victor Chekalin, or Виктор Чекалин in Cyrillic, on formatting a floating point number as a display string using the current project units.
This issue has cropped up several times recently, and various solutions based on the same principle have been suggested, among others by Joe Offord of
[Enclos](http://www.enclos.com),
who already shared insights on accessing
[curtain wall geometry](http://thebuildingcoder.typepad.com/blog/2010/05/curtain-wall-geometry.html),
[speeding up the interactive selection process](http://thebuildingcoder.typepad.com/blog/2010/09/speed-up-selection.html),
[mirroring in a new family and changing the active view](http://thebuildingcoder.typepad.com/blog/2010/11/mirroring-in-a-new-family-and-changing-active-view.html), and
[constructing a planar face transform](http://thebuildingcoder.typepad.com/blog/2011/11/planar-face-transform.html).

All of the solutions to this problem I have seen revolve around stuffing in the value to format into an unused parameter picked up from some arbitrary database element and then calling the AsValueString method on it.
The Parameter class provides this functionality, but unfortunately the API does not include any stand-alone access to it.
Here is Victor's initial query:

**Question:** I need to convert a numeric value to a corresponding display string honouring the current Revit ProjectUnit setting. I cannot find how to do it in the Revit help and began search the answer in your amazing site. I found the
[Unit Conversion](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html) tool
and thought it would fulfil my need, but I was wrong :(

Looking at the Unit Converter code, I discovered that retrieving scale factor to internal units is not easy and you used some trick to get it: you change ProjectUnit, write "1" to family parameter, read value from this parameter, change ProjectUnit back. It works but is really hard and is not an official way.

**Answer:** Try something like this on some otherwise unused length parameter 'p':
```vbnet
  Dim value As String = "=2' + 4'/3"
  Dim t As New Transaction(doc, "Format Length")
  t.Start()
  p.SetValueString(value)
  value = p.AsValueString
  t.RollBack()
  Return value
```

In fact, Joe provided the following helper methods based on this idea implemented in VB:

- StringValueString – Converts a string to AsValueString equivalent.- DblValueString – Converts a double to AsValueString equivalent.- ValueStringDbl – Converts a ValueString to a double.

All three of these methods create and then roll back a temporary transaction to perform their task.

Here is the full implementation of the first of these, StringValueString:
```vbnet
  Public Shared Function StringValueString( \_
    ByVal doc As Document, \_
    ByVal value As String) As String

    ' Locate the arbitrary Length parameter

    Dim p As Parameter \_
      = doc.ProjectInformation.Parameter( \_
      "Parameter Name")

    If p Is Nothing Then
      TaskDialog.Show( \_
        "Revit", \_
        "Missing Project Parameter: Parameter Name")
      Return value
    End If

    Dim tr As New Transaction(doc)
    tr.Start("Format Length")

    Try
      p.SetValueString(value)
      value = p.AsValueString
    Catch ex As Exception

    End Try

    tr.RollBack()

    Return value

  End Function
```

For the second, DblValueString, simply replace the input argument by a floating-point value 'ByVal value As Double' and the two lines to perform the actual conversion by
```csharp
p.Set(value)
sValueString = p.AsValueString
```

Finally, for the third, ValueStringDbl, the input argument 'value' is again a string and the conversion to the floating-point return value is performed by
```csharp
p.SetValueString(value)
dValue = p.AsDouble
```

**Response:** Thanks for the answer.

I wrote some simple extension methods for the Revit API Parameter class to get value in project units and in meters value.
Now it works with Length, Volume and Area (now I don't need any more).
It would take much time to add support for all unit conversions.

Here is the entire implementation of my
[ParameterUnitConverter class](http://pastebin.com/ULHxU95E).
It defines the following methods and data:

- AsProjectUnitTypeDouble – Parameter extension method to retrieve double value parameter in ProjectUnits.- AsMetersValue – Parameter extension method to retrieve double value of parameter in meters unit, i.e. length in meters, area in square meters and volume in cubic meters.- ConvertParameterTypeToUnitType – static method to return the corresponding UnitType for a given ParameterType.- \_map\_parameter\_type\_to\_unit\_type – a dictionary mapping ParameterType enumeration values to the corresponding UnitType ones.

Here is the complete implementation of this:
```python
public static class ParameterUnitConverter
{
  private const double METERS\_IN\_FEET = 0.3048;

  public static double AsProjectUnitTypeDouble(
    this Parameter param )
  {
    if( param.StorageType != StorageType.Double )
      throw new NotSupportedException(
        "Parameter does not have double value" );

    double imperialValue = param.AsDouble();

    Document document = param.Element.Document;

    UnitType ut = ConvertParameterTypeToUnitType(
      param.Definition.ParameterType );

    FormatOptions fo = document.ProjectUnit
      .get\_FormatOptions( ut );

    DisplayUnitType dut = fo.Units;

    // Unit Converter
    // http://www.asknumbers.com

    switch( dut )
    {
      #region Length

      case DisplayUnitType.DUT\_METERS:
        return imperialValue \* METERS\_IN\_FEET; //feet
      case DisplayUnitType.DUT\_CENTIMETERS:
        return imperialValue \* METERS\_IN\_FEET \* 100;
      case DisplayUnitType.DUT\_DECIMAL\_FEET:
        return imperialValue;
      case DisplayUnitType.DUT\_DECIMAL\_INCHES:
        return imperialValue \* 12;
      case DisplayUnitType.DUT\_FEET\_FRACTIONAL\_INCHES:
        NotSupported( dut );
        break;
      case DisplayUnitType.DUT\_FRACTIONAL\_INCHES:
        NotSupported( dut );
        break;
      case DisplayUnitType.DUT\_METERS\_CENTIMETERS:
        return imperialValue \* METERS\_IN\_FEET; //feet
      case DisplayUnitType.DUT\_MILLIMETERS:
        return imperialValue \* METERS\_IN\_FEET \* 1000;

      #endregion // Length

      #region Area

      case DisplayUnitType.DUT\_SQUARE\_FEET:
        return imperialValue;
      case DisplayUnitType.DUT\_ACRES:
        return imperialValue \* 1 / 43560.039;
      case DisplayUnitType.DUT\_HECTARES:
        return imperialValue \* 1 / 107639.104;
      case DisplayUnitType.DUT\_SQUARE\_CENTIMETERS:
        return imperialValue \* Math.Pow( METERS\_IN\_FEET \* 100, 2 );
      case DisplayUnitType.DUT\_SQUARE\_INCHES:
        return imperialValue \* Math.Pow( 12, 2 );
      case DisplayUnitType.DUT\_SQUARE\_METERS:
        return imperialValue \* Math.Pow( METERS\_IN\_FEET, 2 );
      case DisplayUnitType.DUT\_SQUARE\_MILLIMETERS:
        return imperialValue \* Math.Pow( METERS\_IN\_FEET \* 1000, 2 );

      #endregion // Area

      #region Volume
      case DisplayUnitType.DUT\_CUBIC\_FEET:
        return imperialValue;
      case DisplayUnitType.DUT\_CUBIC\_CENTIMETERS:
        return imperialValue \* Math.Pow( METERS\_IN\_FEET \* 100, 3 );
      case DisplayUnitType.DUT\_CUBIC\_INCHES:
        return imperialValue \* Math.Pow( 12, 3 );
      case DisplayUnitType.DUT\_CUBIC\_METERS:
        return imperialValue \* Math.Pow( METERS\_IN\_FEET, 3 );
      case DisplayUnitType.DUT\_CUBIC\_MILLIMETERS:
        return imperialValue \* Math.Pow( METERS\_IN\_FEET \* 1000, 3 );
      case DisplayUnitType.DUT\_CUBIC\_YARDS:
        return imperialValue \* 1 / Math.Pow( 3, 3 );
      case DisplayUnitType.DUT\_GALLONS\_US:
        return imperialValue \* 7.5;
      case DisplayUnitType.DUT\_LITERS:
        return imperialValue \* 28.31684;

      #endregion // Volume

      default:
        NotSupported( dut );
        break;
    }

    throw new NotSupportedException();
  }
  public static double AsMetersValue(
    this Parameter param )
  {
    if( param.StorageType != StorageType.Double )
      throw new NotSupportedException(
        "Parameter does not have double value" );

    double imperialValue = param.AsDouble();

    UnitType ut = ConvertParameterTypeToUnitType(
      param.Definition.ParameterType );

    switch( ut )
    {
      case UnitType.UT\_Length:
        return imperialValue \* METERS\_IN\_FEET;

      case UnitType.UT\_Area:
        return imperialValue \* Math.Pow(
          METERS\_IN\_FEET, 2 );

      case UnitType.UT\_Volume:
        return imperialValue \* Math.Pow(
          METERS\_IN\_FEET, 3 );
    }
    throw new NotSupportedException();
  }

  public static UnitType
    ConvertParameterTypeToUnitType(
      ParameterType parameterType )
  {
    if( \_map\_parameter\_type\_to\_unit\_type.ContainsKey(
      parameterType ) )
    {
      return \_map\_parameter\_type\_to\_unit\_type[
        parameterType];
    }
    else
    {
      // If we made it this far, there's
      // no entry in the dictionary.

      throw new ArgumentException(
        "Cannot convert ParameterType '"
          + parameterType.ToString()
          + "' to a UnitType." );
    }
  }

  static Dictionary<ParameterType, UnitType>
    \_map\_parameter\_type\_to\_unit\_type
      = new Dictionary<ParameterType, UnitType>()
  {
    // This data could come from a file,
    // or (better yet) from Revit itself...

    {ParameterType.Angle, UnitType.UT\_Angle},
    {ParameterType.Area, UnitType.UT\_Area},
    {ParameterType.AreaForce, UnitType.UT\_AreaForce},
    {ParameterType.AreaForcePerLength, UnitType.UT\_AreaForcePerLength},
    //map.Add(UnitType.UT\_AreaForceScale, ParameterType.???);
    {ParameterType.ColorTemperature, UnitType.UT\_Color\_Temperature},
    {ParameterType.Currency, UnitType.UT\_Currency},
    //map.Add(UnitType.UT\_DecSheetLength, ParameterType.???);
    {ParameterType.ElectricalApparentPower, UnitType.UT\_Electrical\_Apparent\_Power},
    {ParameterType.ElectricalCurrent, UnitType.UT\_Electrical\_Current},
    {ParameterType.ElectricalEfficacy, UnitType.UT\_Electrical\_Efficacy},
    {ParameterType.ElectricalFrequency, UnitType.UT\_Electrical\_Frequency},
    {ParameterType.ElectricalIlluminance, UnitType.UT\_Electrical\_Illuminance},
    {ParameterType.ElectricalLuminance, UnitType.UT\_Electrical\_Luminance},
    {ParameterType.ElectricalLuminousFlux, UnitType.UT\_Electrical\_Luminous\_Flux},
    {ParameterType.ElectricalLuminousIntensity, UnitType.UT\_Electrical\_Luminous\_Intensity},
    {ParameterType.ElectricalPotential, UnitType.UT\_Electrical\_Potential},
    {ParameterType.ElectricalPower, UnitType.UT\_Electrical\_Power},
    {ParameterType.ElectricalPowerDensity, UnitType.UT\_Electrical\_Power\_Density},
    {ParameterType.ElectricalWattage, UnitType.UT\_Electrical\_Wattage},
    {ParameterType.Force, UnitType.UT\_Force},
    {ParameterType.ForceLengthPerAngle, UnitType.UT\_ForceLengthPerAngle},
    {ParameterType.ForcePerLength, UnitType.UT\_ForcePerLength},
    //map.Add(UnitType.UT\_ForceScale, ParameterType.???);
    {ParameterType.HVACAirflow, UnitType.UT\_HVAC\_Airflow},
    {ParameterType.HVACAirflowDensity, UnitType.UT\_HVAC\_Airflow\_Density},
    {ParameterType.HVACAirflowDividedByCoolingLoad, UnitType.UT\_HVAC\_Airflow\_Divided\_By\_Cooling\_Load},
    {ParameterType.HVACAirflowDividedByVolume, UnitType.UT\_HVAC\_Airflow\_Divided\_By\_Volume},
    {ParameterType.HVACAreaDividedByCoolingLoad, UnitType.UT\_HVAC\_Area\_Divided\_By\_Cooling\_Load},
    {ParameterType.HVACAreaDividedByHeatingLoad, UnitType.UT\_HVAC\_Area\_Divided\_By\_Heating\_Load},
    {ParameterType.HVACCoefficientOfHeatTransfer, UnitType.UT\_HVAC\_CoefficientOfHeatTransfer},
    {ParameterType.HVACCoolingLoad, UnitType.UT\_HVAC\_Cooling\_Load},
    {ParameterType.HVACCoolingLoadDividedByArea, UnitType.UT\_HVAC\_Cooling\_Load\_Divided\_By\_Area},
    {ParameterType.HVACCoolingLoadDividedByVolume, UnitType.UT\_HVAC\_Cooling\_Load\_Divided\_By\_Volume},
    {ParameterType.HVACCrossSection, UnitType.UT\_HVAC\_CrossSection},
    {ParameterType.HVACDensity, UnitType.UT\_HVAC\_Density},
    {ParameterType.HVACDuctSize, UnitType.UT\_HVAC\_DuctSize},
    {ParameterType.HVACEnergy, UnitType.UT\_HVAC\_Energy},
    {ParameterType.HVACFactor, UnitType.UT\_HVAC\_Factor},
    {ParameterType.HVACFriction, UnitType.UT\_HVAC\_Friction},
    {ParameterType.HVACHeatGain, UnitType.UT\_HVAC\_HeatGain},
    {ParameterType.HVACHeatingLoad, UnitType.UT\_HVAC\_Heating\_Load},
    {ParameterType.HVACHeatingLoadDividedByArea, UnitType.UT\_HVAC\_Heating\_Load\_Divided\_By\_Area},
    {ParameterType.HVACHeatingLoadDividedByVolume, UnitType.UT\_HVAC\_Heating\_Load\_Divided\_By\_Volume},
    {ParameterType.HVACPower, UnitType.UT\_HVAC\_Power},
    {ParameterType.HVACPowerDensity, UnitType.UT\_HVAC\_Power\_Density},
    {ParameterType.HVACPressure, UnitType.UT\_HVAC\_Pressure},
    {ParameterType.HVACRoughness, UnitType.UT\_HVAC\_Roughness},
    {ParameterType.HVACSlope, UnitType.UT\_HVAC\_Slope},
    {ParameterType.HVACTemperature, UnitType.UT\_HVAC\_Temperature},
    {ParameterType.HVACVelocity, UnitType.UT\_HVAC\_Velocity},
    {ParameterType.HVACViscosity, UnitType.UT\_HVAC\_Viscosity},
    {ParameterType.Length, UnitType.UT\_Length},
    {ParameterType.LinearForce, UnitType.UT\_LinearForce},
    {ParameterType.LinearForceLengthPerAngle, UnitType.UT\_LinearForceLengthPerAngle},
    {ParameterType.LinearForcePerLength, UnitType.UT\_LinearForcePerLength},
    // map.Add(UnitType.UT\_LinearForceScale, ParameterType.???);
    {ParameterType.LinearMoment, UnitType.UT\_LinearMoment},
    // map.Add(UnitType.UT\_LinearMomentScale, ParameterType.???);
    {ParameterType.Moment, UnitType.UT\_Moment},
    ///map.Add(UnitType.UT\_MomentScale, ParameterType.???);
    {ParameterType.Number, UnitType.UT\_Number},
    {ParameterType.PipeSize, UnitType.UT\_PipeSize},
    {ParameterType.PipingDensity, UnitType.UT\_Piping\_Density},
    {ParameterType.PipingFlow, UnitType.UT\_Piping\_Flow},
    {ParameterType.PipingFriction, UnitType.UT\_Piping\_Friction},
    {ParameterType.PipingPressure, UnitType.UT\_Piping\_Pressure},
    {ParameterType.PipingRoughness, UnitType.UT\_Piping\_Roughness},
    {ParameterType.PipingSlope, UnitType.UT\_Piping\_Slope},
    {ParameterType.PipingTemperature, UnitType.UT\_Piping\_Temperature},
    {ParameterType.PipingVelocity, UnitType.UT\_Piping\_Velocity},
    {ParameterType.PipingViscosity, UnitType.UT\_Piping\_Viscosity},
    {ParameterType.PipingVolume, UnitType.UT\_Piping\_Volume},
    //map.Add(UnitType.UT\_SheetLength, ParameterType.???);
    //map.Add(UnitType.UT\_SiteAngle, ParameterType.???);
    {ParameterType.Slope, UnitType.UT\_Slope},
    {ParameterType.Stress, UnitType.UT\_Stress},
    {ParameterType.TemperalExp, UnitType.UT\_TemperalExp},
    {ParameterType.UnitWeight, UnitType.UT\_UnitWeight},
    {ParameterType.Volume, UnitType.UT\_Volume},
    {ParameterType.WireSize, UnitType.UT\_WireSize},
  };
}
```

I used some functions from the Unit converter.
Here is an external command
[sample to test it](http://pastebin.com/Pu26SPAN) by
iterating over and applying it to all floating point valued parameters on a selected element.

Hope you'll find this useful.

Many thanks to Joe and Victor for putting together and sharing these nice solutions!

I added the ParameterUnitConverter class to The Building Coder samples and defined a new external command CmdParameterUnitConverter based on Victor's code to test it.
Here is
[version 2012.0.96.0](zip/bc_12_96.zip) of
The Building Coder samples including the new utility class and command.

This is the output generated by the command in the Visual Studio debug output window on selecting a wall element in the rac\_basic\_sample\_project.rvt sample model:

```
Parameter name: Top Extension Distance
  Parameter value (imperial): 0
  Parameter unit value: 0
  Parameter AsValueString: 0.0

Parameter name: Length
  Parameter value (imperial): 45.5
  Parameter unit value: 13868.4
  Parameter AsValueString: 13868.4

Parameter name: Base Extension Distance
  Parameter value (imperial): 0
  Parameter unit value: 0
  Parameter AsValueString: 0.0

Parameter name: Top Offset
  Parameter value (imperial): 0
  Parameter unit value: 0
  Parameter AsValueString: 0.0

Parameter name: Volume
  Parameter value (imperial): 455.9586023831
  Parameter unit value: 12.911309795985
  Parameter AsValueString: 12.911 m³

Parameter name: Unconnected Height
  Parameter value (imperial): 18.0446194225722
  Parameter unit value: 5500
  Parameter AsValueString: 5500.0

Parameter name: Base Offset
  Parameter value (imperial): 0
  Parameter unit value: 0
  Parameter AsValueString: 0.0

Parameter name: Area
  Parameter value (imperial): 694.880910031848
  Parameter unit value: 64.5565489799251
  Parameter AsValueString: 64.557 m²
```

Here, the parameter value labelled 'imperial' is the
[internal Revit database unit](http://thebuildingcoder.typepad.com/blog/2011/01/unit-conversion-and-new-blogs.html), e.g.
[feet for length](http://thebuildingcoder.typepad.com/blog/2011/03/internal-imperial-units.html).
Please note that not all internal database units are imperial.
In fact, only length is measured in feet, and thus also area and volume.
Other internal units are
[SI-based](http://en.wikipedia.org/wiki/International_System_of_Units).