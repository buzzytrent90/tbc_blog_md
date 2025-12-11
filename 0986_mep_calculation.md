---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.056224'
original_url: https://thebuildingcoder.typepad.com/blog/0986_mep_calculation.html
post_number: 0986
reading_time_minutes: 4
series: mep
slug: mep_calculation
source_file: 0986_mep_calculation.htm
tags:
- csharp
- revit-api
- schedules
- mep
title: User MEP Calculation Sample
word_count: 853
---

### User MEP Calculation Sample

One of the main new features in the Revit 2014 API is the possibility to make use of external services to redefine the algorithms used for certain MEP related calculations.

The
[external services framework](http://thebuildingcoder.typepad.com/blog/2012/05/the-revit-2013-mep-api-and-external-services.html#3) was introduced in Revit 2013, but
[not used](http://thebuildingcoder.typepad.com/blog/2012/08/updated-revit-mep-2013-material.html#7) in
that version.

The recent listing of
[Revit 2014 API functionality and SDK samples](http://thebuildingcoder.typepad.com/blog/2013/07/revit-2014-obj-exporter-and-new-sdk-samples.html#3) points
out that this is one of the highlights of the new API, and yet no sample using it has been published yet.

Let's rectify that right here and now.

#### Built-in External Services

Revit MEP 2014 makes use of the external services itself.
The service implementations live in the MEPCalculation sub-folder of 'C:\Program Files\Autodesk\Revit 2014\AddIns'.

It contains the following add-in manifests and .NET assemblies, with the DLLs listed under the add-in manifests loading them:

- FittingAndAccessoryCalculationManaged.dll
- FittingAndAccessoryCalculationUIServers.addin

- FittingAndAccessoryCalculationUIServers.dll

- MEPCalculation.addin

- FittingAndAccessoryCalculationServers.dll
- StraightSegmentCalculationServers.dll

- PressureLossReport.addin

- PressureLossReport.dll

A very similar structure was extracted and released in a pre-release API sample on the Revit beta site as an example of implementing a custom external service, but never made it into the final release.

Prompted and supported by our MEP expert Martin Schmid, I now took a look at that and adapted it to the current Revit 2014 API.

Here are the steps and results:

- [The UserMepCalculation sample](#2)
- [Concepts and Use Cases](#3)
- [Migration of the pre-alpha version](#4)
- [Test run](#5)
- [Download](#6)

#### The UserMepCalculation Sample

The UserMepCalculation add-in is an external application implementing user-defined MEP calculation solver external services that override Revit’s default MEPCalculation solvers and reports listed above, in particular:

- System pressure loss report
- Straight segment calculation
- Fitting and accessory calculation

The UserMepCalculation Visual Studio solution contains the following four C# projects:

- FittingAndAccessoryCalculationServers
- FittingAndAccessoryCalculationUIServers
- PressureLossReport
- StraightSegmentCalculationServers

#### Concepts and Use Cases

Actually, digging in deeper, the sample addresses these six separate concepts:

1. Pipe segment pressure loss calculation
2. Pipe fitting/accessory pressure loss calculation
3. Pipe fixture units to volume flow conversion calculation
4. Duct segment pressure loss calculation
5. Duct fitting/accessory pressure loss calculation
6. Pressure loss report

#### Migration of the Pre-alpha Version

Compiling the original pre-alpha version of this add-in produced the following initial list of [3 errors and 6 warnings](zip/mep_calculations_migr_a.txt).

I applied the
[disable architecture mismatch warning utility DisableMismatchWarning.exe](http://thebuildingcoder.typepad.com/blog/2013/07/recursively-disable-architecture-mismatch-warning.html), removing the warnings, leaving just [3 errors](zip/mep_calculations_migr_b.txt).

All three are similar, complaining that the three server implementations for pipe plumbing fixture flow and duct and pipe pressure drop, derived from the three interfaces IPipePressureDropServer, IDuctPressureDropServer and IPipePlumbingFixtureFlowServer, are not fulfilling their contract that requires them to implement the GetHtmlDescription member method.

Once the three methods were added, the compilation proceeded one step further and reported the next [4 errors](zip/mep_calculations_migr_c.txt), all referring to an erroneous call to an InnerDiameter method.
That property was since renamed to InsideDiameter.
After fixing that as well, the sample builds with zero errors and warnings.

It makes use of some unnecessary internal project settings that I cleaned up and removed as far as possible.
Some remnants are still left, though.
Among other things, the projects are set up to build specific 32 and 64 bit versions, although they are all identical, being standard .NET IL assemblies.

Once this was cleaned up, compiled and installed, it could be tested.

#### Test Run

To use and test, build and install the add-in.
Note that the new user defined alternate calculation methods are now available and can be selected via Manage > MEP Settings > Mechanical Settings > Duct Settings and Pipe Settings > Calculation.

User duct pressure drop:

![User duct pressure drop](img/mep_calc_user_duct_pressure_drop.png)

User pipe pressure drop:

![User pipe pressure drop](img/mep_calc_user_pipe_pressure_drop.png)

User plumbing fixture flow:

![User plumbing fixture flow](img/mep_calc_user_plumbing_fixture_flow.png)

Let's examine a system pressure-loss report.
Note that the project-defined user interface now appears after launching Analyze > Reports & Schedules > Pipe Pressure Loss Report.

I copied the default pressure loss report transformation PressureLossReport.xslt from the above-mentioned MEPCalculation folder and renamed it to UserPressureLossReport.xslt.

The standard Revit pressure loss command now picks up my redefined calculation rules and created this
[pressure loss report](zip/jeremy.html) from
the Revit MEP basic sample project rme\_basic\_sample\_project.rvt.

![Pressure loss report](img/mep_calc_user_pressure_loss_report.png)

#### Download

Here is
[UserMepCalculation.zip](zip/UserMepCalculation.zip) containing
the complete source code, Visual Studio solution, add-in manifest and some documentation on the custom calculation and report add-in.

I am off to Australia Sunday evening to hold a Revit API training there next week, and want to do some climbs in the Furka pass this weekend before leaving, so wish me luck!