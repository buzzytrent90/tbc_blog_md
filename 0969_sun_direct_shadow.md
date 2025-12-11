---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.015387'
original_url: https://thebuildingcoder.typepad.com/blog/0969_sun_direct_shadow.html
post_number: 0969
reading_time_minutes: 4
series: general
slug: sun_direct_shadow
source_file: 0969_sun_direct_shadow.htm
tags:
- csharp
- elements
- geometry
- revit-api
- vbnet
- views
title: Sun Direction, Shadow Calculation and Wizard Update
word_count: 708
---

### Sun Direction, Shadow Calculation and Wizard Update

I enjoyed some wonderful hot sunny days since I returned from New England back to Switzerland.

Summer finally arrived.
A nice change after the rain and cold both here and in New Hampshire and Massachusetts.

Talking about rain and sun leads up nicely to today's topic, on the sun direction and shadow calculation in a Revit BIM.

Before I get to that, though, I present an update of my Revit add-in wizards resulting in part from yesterday's post on the
[processor architecture mismatch warning](http://thebuildingcoder.typepad.com/blog/2013/06/processor-architecture-mismatch-warning.html):

- [Visual Studio Revit add-in wizard update](#2)
- [Calculating and marking shaded areas](#3)
- [Determining sun direction](#4)

#### Visual Studio Revit Add-in Wizard Update

I implemented updates of my
[Visual Studio wizards to generate new C# and VB Revit 2014 add-ins](http://thebuildingcoder.typepad.com/blog/2013/05/add-in-wizards-for-revit-2014-1.html) yesterday,
for two reasons:

- Suppress the
  [processor architecture mismatch warning](http://thebuildingcoder.typepad.com/blog/2013/06/processor-architecture-mismatch-warning.html) by
  adding a property group to the Visual Studio project file – I always try to compile all my projects with zero warnings.
- Modified the Revit executable and API assembly paths to refer to Revit Onebox instead of the architectural flavour – the former was unavailable when I first installed Revit 2014.

To install, simply copy the zip file of your choice to the matching Visual Studio project template folder in your local file system:

- [Revit2014AddinWizardCs2.zip](zip/Revit2014AddinWizardCs2.zip) – copy to

  [My Documents]\Visual Studio 2010\Templates\ProjectTemplates\Visual C#
- [Revit2014AddinWizardVb2.zip](zip/Revit2014AddinWizardVb2.zip) – copy to

  [My Documents]\Visual Studio 2010\Templates\ProjectTemplates\Visual Basic

For all further details on this, links back to previous versions, other flavours, and tips on how to adapt the wizards for your own use, please refer to the
[initial 2014 version](http://thebuildingcoder.typepad.com/blog/2013/05/add-in-wizards-for-revit-2014-1.html).

#### Calculating and Marking Shaded Areas

Determining the shaded areas in a model is obviously important for many areas of analysis.

Assuming certain objects in the BIM cast a shade over others, is there a way to programmatically determine the shadowed areas?

In fact, the Revit API provides a dedicated ExtrusionAnalyzer class for this very kind of calculation, which I made use of to determine the
[plan view furniture and equipment boundary loops](http://thebuildingcoder.typepad.com/blog/2013/04/extrusion-analyser-and-plan-view-boundaries.html) for
my 2D cloud-based editor workflow.

Is there a way to mark these areas in the model?

That sounds rather complicated.
You might be able to use
[Boolean operations for 2D polygons](http://thebuildingcoder.typepad.com/blog/2009/02/boolean-operations-for-2d-polygons.html) and
paint the shaded areas.

#### Calculating Sun Direction for Shadow Calculation

This leads to the following

**Question:** I found code snippet showing how to use the ExtrusionAnalyzer class to calculate shadows in the material provided for the Autodesk University class on *Geometric Progression: Further Analysis of Geometry using Autodesk Revit 2012 API*.

To make use of it, I also need to determine the sun direction using the Revit API.
It obviously differs based on time of the day, location, etc.

How can I obtain the sun direction for any given project location, date and time, please?

**Answer:** The sun direction is available from the Sun and Shadows element.
Of course, it always represents a single point in time (date, time) and not the complete range of possible sun directions.
The sample takes this into account and recalculates shadows when the sun is moved to another date and time location via the user interface.

Here is the code to determine the sun direction from the sun and shadows settings:
```csharp
public class ShadowCalculatorUtils
{
  public static XYZ GetSunDirection( View view )
  {
    SunAndShadowSettings sunSettings
      = view.SunAndShadowSettings;

    XYZ initialDirection = XYZ.BasisY;

    //double altitude = sunSettings.Altitude;

    double altitude = sunSettings.GetFrameAltitude(
      sunSettings.ActiveFrame );

    Transform altitudeRotation = Transform
      .CreateRotation( XYZ.BasisX, altitude );

    XYZ altitudeDirection = altitudeRotation
      .OfVector( initialDirection );

    //double azimuth = sunSettings.Azimuth;

    double azimuth = sunSettings.GetFrameAzimuth(
      sunSettings.ActiveFrame );

    double actualAzimuth = 2 \* Math.PI - azimuth;

    Transform azimuthRotation = Transform
      .CreateRotation( XYZ.BasisZ, actualAzimuth );

    XYZ sunDirection = azimuthRotation.OfVector(
      altitudeDirection );

    return sunDirection;
  }
}
```