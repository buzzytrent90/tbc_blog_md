---
post_number: "0011"
title: "Units"
slug: "units"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'revit-api', 'views', 'walls', 'windows']
source_file: "0011_units.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0011_units.html"
---

### Units

In the
[previous post](http://thebuildingcoder.typepad.com/blog/2008/09/selecting-all-w.html),
we noted that the area parameter for a wall had a numerical value of 283.65 on one hand, but a string representation of '26.352 m' on the other. The reason for these two different values is that Revit internally works with a fixed set of database units for different measurement types. These internal database units cannot be changed. All raw database values obtained from attributes will always be returned and expected in database units. Some of the internal database length units are imperial. The database unit for length is feet, and areas are stored in square feet.
The units used are not the standardised
[SI units](http://nostalgia.wikipedia.org/wiki/SI_unit)
according to the International System of Units SI.

Similarly, if we examine the thickness of a 200 mm wall, e.g. one using the system family 'Basic Wall' and the type 'Generic - 200.0 mm', its thickness is available in the type parameter 'width'. It appears to have the value 200 mm in the user interface, if set up to display metric length units. Looking at the actual value of the underlying parameter, however, returns a completely different number, something like 0.6561. 0.6561 feet is 0.6561 \* 12 = 7.8732 inches, which corresponds to 7.8732 \* 25.4 = 199.97928 millimetres.

The user interface settings affecting the display of units in Revit can be queried through the API. The ProjectUnit SDK sample demonstrates how to do this.
The units displayed in the user interface can be queried through the read-only Document.DisplayUnitSystem property. The Parameter.DisplayUnitType property determines how any individual parameter is displayed in the user interface. The DisplayUnitType enumeration lists all the supported display unit types.

It can sometimes be tricky to figure out exactly what units Revit is using internally, for instance when analysing things like point, line and area loads in Revit Structure.
For these loads, the units used are in terms of N and m, unlike other parameters and co-ordinates which use feet for all lengths.

The good news, and the main message of this post, is that you do not really have to care what units Revit is using internally. All you need to do is some little research to determine a valid conversion factor. You can enter the unit value for whatever you are interested in in the user interface, and then check what the API reports. It will always be in the same internal units. This can then be used as a conversion factor for that specific unit. It works consistently. Possibly you may refine this depending on the metric or imperial Revit setting, but the factors are the same and consistent.

So the simplest thing to do to determine the conversion factor to use is to set the parameter to 1 in the user interface and then observe the value returned by the API, which gives the conversion factor between the two.

I am appending a small utility class UnitConversion.cs which may be of use in this context, even if just to give you a first impression of how these issues may be handled. I am not saying that this is the optimal solution. It includes some useful constants and stems from the Midas Link Revit Structure sample application which is available including source code on the ADN web site. By the way, the updated MidasLink version for Revit 2009 has just been posted to ADN, in case you are interested. Please excuse the fact that the source code lines exceed the blog display width. To obtain and review the full source code, please simply copy and paste it to notepad or some other editor.

```csharp
// (C) Copyright 2003-2007 by Autodesk, Inc.
//
// Permission to use, copy, modify, and distribute this software in
// object code form for any purpose and without fee is hereby granted,
// provided that the above copyright notice appears in all copies and
// that both that copyright notice and the limited warranty and
// restricted rights notice below appear in all supporting
// documentation.
//
// AUTODESK PROVIDES THIS PROGRAM "AS IS" AND WITH ALL FAULTS.
// AUTODESK SPECIFICALLY DISCLAIMS ANY IMPLIED WARRANTY OF
// MERCHANTABILITY OR FITNESS FOR A PARTICULAR USE. AUTODESK, INC.
// DOES NOT WARRANT THAT THE OPERATION OF THE PROGRAM WILL BE
// UNINTERRUPTED OR ERROR FREE.
//
// Use, duplication, or disclosure by the U.S. Government is subject to
// restrictions set forth in FAR 52.227-19 (Commercial Computer
// Software - Restricted Rights) and DFAR 252.227-7013(c)(1)(ii)
// (Rights in Technical Data and Computer Software), as applicable.
//

using System;
using System.Collections.Generic;
using System.Text;
using System.Collections;
using System.Windows.Forms;
using System.Diagnostics;

using Autodesk.Revit;
using Autodesk.Revit.Elements;
using Autodesk.Revit.Enums;
using Autodesk.Revit.Parameters;

namespace Revit.MGT
{
  /// <summary>
  /// Unit conversion - convert values from standard units to given units.
  /// </summary>
  public class UnitConversionFactors
  {
    #region Class Member Variables

    /// <summary>
    /// Standard units
    /// </summary>
    private string m\_FromUnits;

    /// <summary>
    /// Given units
    /// </summary>
    private string m\_ToUnits;

    /// <summary>
    /// true: conversion available for given units
    /// </summary>
    private bool m\_RatioSetUpSucceeded;

    /// <summary>
    /// To store length ratio
    /// </summary>
    private double m\_LengthRatio;

    /// <summary>
    /// To store point load ratio
    /// </summary>
    private double m\_PointLoadRatio;

    /// <summary>
    /// To store point load moment ratio
    /// </summary>
    private double m\_PointLoadMomentRatio;

    /// <summary>
    /// To store line load ratio
    /// </summary>
    private double m\_LineLoadRatio;

    /// <summary>
    /// To store line moment ratio
    /// </summary>
    private double m\_LineMomentRatio;

    /// <summary>
    /// To store area load ratio
    /// </summary>
    private double m\_AreaLoadRatio;

    /// <summary>
    /// To store stress ratio
    /// </summary>
    private double m\_StressRatio;

    /// <summary>
    /// To store unit weight ratio
    /// </summary>
    private double m\_UnitWeightRatio;

    /// <summary>
    /// To store point spring ratio
    /// </summary>
    private double m\_PointSpringRatio;

    /// <summary>
    /// To store rotational point spring ratio
    /// </summary>
    private double m\_RotationalPointSpringRatio;

    #endregion

    #region Class Public Properties
    /// <summary>
    /// Get FromUnits
    /// </summary>
    public string FromUnits
    {
      get
      {
        return m\_FromUnits;
      }
    }

    /// <summary>
    /// Get ToUnits
    /// </summary>
    public string ToUnits
    {
      get
      {
        return m\_ToUnits;
      }
    }

    /// <summary>
    /// Get RatioSetUpSucceeded
    /// </summary>
    public bool RatioSetUpSucceeded
    {
      get
      {
        return m\_RatioSetUpSucceeded;
      }
    }

    /// <summary>
    /// Get length ratio
    /// </summary>
    public double LengthRatio
    {
      get
      {
        return m\_LengthRatio;
      }
    }

    /// <summary>
    /// Get point load ratio
    /// </summary>
    public double PointLoadRatio
    {
      get
      {
        return m\_PointLoadRatio;
      }
    }

    /// <summary>
    /// Get point load moment ratio
    /// </summary>
    public double PointLoadMomentRatio
    {
      get
      {
        return m\_PointLoadMomentRatio;
      }
    }

    /// <summary>
    /// Get line load force ratio
    /// </summary>
    public double LineLoadForceRatio
    {
      get
      {
        return m\_LineLoadRatio;
      }
    }

    /// <summary>
    /// Get line load moment ratio
    /// </summary>
    public double LineLoadMomentRatio
    {
      get
      {
        return m\_LineMomentRatio;
      }
    }

    /// <summary>
    /// Get area load ratio
    /// </summary>
    public double AreaLoadRatio
    {
      get
      {
        return m\_AreaLoadRatio;
      }
    }

    /// <summary>
    /// Get stress ratio
    /// </summary>
    public double StressRatio
    {
      get
      {
        return m\_StressRatio;
      }
    }

    /// <summary>
    /// Get unit weight ratio
    /// </summary>
    public double UnitWeightRatio
    {
      get
      {
        return m\_UnitWeightRatio;
      }
    }

    /// <summary>
    /// Get point sprint ratio
    /// </summary>
    public double PointSpringRatio
    {
      get
      {
        return m\_PointSpringRatio;
      }
    }

    /// <summary>
    /// Get rotational point spring ratio
    /// </summary>
    public double RotationalPointSpringRatio
    {
      get
      {
        return m\_RotationalPointSpringRatio;
      }
    }
    #endregion

    #region Class Constructor
    /// <summary>
    /// Constructor
    /// </summary>
    /// <param name="lengthUnit">length unit</param>
    /// <param name="forceUnit">forth unit</param>
    public UnitConversionFactors(
      string lengthUnit,
      string forceUnit )
    {
      // Initialize standard factors
      InitializeConversionFactors();

      // Get length conversion factor
      double lenFactor = GetLengthConversionFactor(lengthUnit);
      if (!m\_RatioSetUpSucceeded)
      {
        return;
      }

      // Get Force conversion factor
      double forceFactor = GetForceConversionFactor(forceUnit);
      if (!m\_RatioSetUpSucceeded)
      {
        return;
      }

      // Set up all the conversion factors
      m\_ToUnits = lengthUnit + '-' + forceUnit;
      SetUnitConversionFactors(lenFactor, forceFactor);
    }
    #endregion

    #region Class Implementation
    /// <summary>
    /// Initialize factor to convert internal units to standard ft, kips.
    /// </summary>
    private void InitializeConversionFactors()
    {
      // Internally Revit stores length in feet and other quantities in metric units.
      // Thus the derived unit for force is stored in a non-standard unit: kg-ft/s\*\*2.
      // For example, m\_PointLoadRatio below equals 1 (kip) / 14593.90 (kg-ft/s\*\*2)
      m\_FromUnits = "ft-kips";
      m\_ToUnits = "";
      m\_LengthRatio = 1;
      m\_PointLoadRatio = 0.00006852176585679176;
      m\_PointLoadMomentRatio = 0.00006852176585679176;
      m\_LineLoadRatio = 0.00006852176585679176;
      m\_LineMomentRatio = 0.00006852176585679176;
      m\_AreaLoadRatio = 0.00006852176585679176;
      m\_StressRatio = 0.00006852176585679176;
      m\_UnitWeightRatio = 0.00006852176585679176;
      m\_PointSpringRatio = 0.00006852176585679176;
      m\_RotationalPointSpringRatio = 0.00006852176585679176 \* 180 / Math.PI;  // Revit uses degrees, Midas uses radians
      m\_RatioSetUpSucceeded = false;
    }

    /// <summary>
    /// Get length conversation factor
    /// </summary>
    /// <param name="lengthUnit">length unit type</param>
    /// <returns>length conversation factor</returns>
    private double GetLengthConversionFactor(
      string lengthUnit )
    {
      bool unitAvailable = true;
      double lenFac = 0;

      switch (lengthUnit)
      {
        case "ft":
          lenFac = 1;
          break;
        case "in":
          lenFac = 12;
          break;
        case "m":
          lenFac = 0.3048;
          break;
        case "cm":
          lenFac = 30.48;
          break;
        case "mm":
          lenFac = 304.8;
          break;
        default:
          unitAvailable = false;
          break;
      }
      m\_RatioSetUpSucceeded = unitAvailable;
      return lenFac;
    }

    /// <summary>
    /// Get force conversation factor
    /// </summary>
    /// <param name="forceUnit">force unit</param>
    /// <returns>force conversation factor</returns>
    private double GetForceConversionFactor(
      string forceUnit )
    {
      bool unitAvailable = true;
      double forceFac = 0;

      switch (forceUnit)
      {
        case "kips":
          forceFac = 1;
          break;
        case "lbf":
          forceFac = 1000;
          break;
        case "kN":
          forceFac = 4.4482216152605;
          break;
        case "N":
          forceFac = 4448.2216152605;
          break;
        case "tonf": // metric tonne
          forceFac = 4.4482216152605 \* 0.101971999794;
          break;
        case "kgf":
          forceFac = 4448.2216152605 \* 0.101971999794;
          break;
        default:
          unitAvailable = false;
          break;
      }
      m\_RatioSetUpSucceeded = unitAvailable;
      return forceFac;
    }

    /// <summary>
    /// Set length and force unit conversation factor
    /// </summary>
    /// <param name="lenFac">length unit factor</param>
    /// <param name="forceFac">force unit factor</param>
    private void SetUnitConversionFactors(
      double lenFac,
      double forceFac )
    {
      m\_LengthRatio \*= (lenFac);
      m\_PointLoadRatio \*= (forceFac);
      m\_PointLoadMomentRatio \*= (forceFac \* lenFac);
      m\_LineLoadRatio \*= (forceFac / lenFac);
      m\_LineMomentRatio \*= (forceFac);
      m\_AreaLoadRatio \*= (forceFac / lenFac / lenFac);
      m\_StressRatio \*= (forceFac / lenFac / lenFac);
      m\_UnitWeightRatio \*= (forceFac / lenFac / lenFac / lenFac);
      m\_PointSpringRatio \*= (forceFac / lenFac);
      m\_RotationalPointSpringRatio \*= (forceFac \* lenFac);
    }
    #endregion
  }
}
```