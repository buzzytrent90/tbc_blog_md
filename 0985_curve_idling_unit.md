---
post_number: "0985"
title: "Curve Length, Idling, Units and RevitPythonShell"
slug: "curve_idling_unit"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'geometry', 'parameters', 'python', 'revit-api', 'transactions', 'walls']
source_file: "0985_curve_idling_unit.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0985_curve_idling_unit.html"
---

### Curve Length, Idling, Units and RevitPythonShell

Here are a couple of recent interesting questions that came up:

- [Curve Length versus ApproximateLength](#2)
- [Potential SetRaiseWithoutDelay workaround](#3)
- [Unit handling improvements](#4)
- [Self-contained RevitPythonShell deployment](#5)

#### Curve Length versus ApproximateLength

**Question:** What is the difference between the Curve Length and ApproximateLength properties, please?

I am using them to obtain an arc wall length.
As far as I can tell, the result is exactly the same, so which one should I normally use?

**Answer:** All Revit geometry
[curves are parameterised](http://thebuildingcoder.typepad.com/blog/2010/01/curve-parameterisation.html).

A curve is a mapping from a simple one-dimensional parameter range, e.g. the interval [0,1] or [0,2π], to a set of contiguous points in 2D or 3D space.

Whereas the parameter range is normally very simple, the resulting curve points in space are a different matter.

For a complex curve, the range spanned by the points and the resulting curve length may be difficult to determine exactly.
The Length property may calculate it using numeric integration, which provides a pretty good approximation and may be expensive to compute.

The ApproximateLength property uses a simpler, faster, more efficient algorithm to determine an approximate length without integration.

However, as stated in the documentation, the results may be off by a significant amount.

For arcs and lines there is no difference, as you noted.
In those simple cases, the result and performance for both will always be identical.

Therefore, for walls, under normal circumstances, it will make no difference which one of the two methods you use.

The only difference will occur for complex curves, in which case you need to decide for yourself whether speed or precision is of greater importance for you.

If you have a plan requiring a wall that returns different results for these two properties, your architect is probably mad. :-)

#### Potential SetRaiseWithoutDelay Workaround

Revit 2014 introduced a new
[default Idling behaviour](http://thebuildingcoder.typepad.com/blog/2012/04/idling-enhancements-and-external-events.html#2),
raising the event one single time each time Revit enters an idle session, and the SetRaiseWithoutDelay method to restore the non-default mode, calling the event handler repeatedly.

Calling SetRaiseWithoutDelay may cause more calls than needed and degrade system performance.

However, not calling it means that your Idling event handler will be called once only when Revit enters the idle state, and never again unless the user moves the mouse or takes some other action.

Joe Ye presented a
[trick to force trigger Idling event](http://adndevblog.typepad.com/aec/2013/07/tricks-to-force-trigger-idling-event.html) that
can be used to achieve something in between the two extremes by simulating a mouse move, triggering a new Idling call without calling SetRaiseWithoutDelay.

It may or may not be possible to combine this trick with a timer to achieve any desired delay between the calls.

#### Unit Handling Improvements

**Question:** My add-in needs to create a list of all DisplayUnitTypes (e.g. DUT\_DECIMAL\_INCHES, DUT\_FEET\_FRACTIONAL\_INCHES) that are valid for a given UnitType (e.g. UT\_Length) in a family document.

The only way I could find to do this previously was to try applying every DisplayUnitType enum value to the FormatOptions.Units object for the given UnitType.
The invalid ones throw an exception, so the rest are presumably valid.

Since this is a time-consuming process, I cache the results for all UnitTypes.

The Revit 2014 API introduced the FormatOptions.DisplayUnits property.
The Visual Studio IntelliSense says it can throw three exceptions: ArgumentException, ArgumentOutOfRangeException and InvalidOperationException.

However, in all my tests so far, the method **never** throws an exception.
My add-in therefore concludes that every possible DisplayUnitType is valid for the FormatOptions for every UnitType.

I know that significant changes were made to the Units engine.

Is there any better built-in way to generate the list of DisplayUnitTypes that are valid for a UnitType?

**Answer:** This is working as designed:

FormatOptions.DisplayUnits rejects only "invalid display units".
The documentation refers to UnitUtils.GetValidDisplayUnits, which says, "A display unit is considered valid if it is an actual unit like meters or feet. Obsolete values and the special values DUT\_CUSTOM and DUT\_UNDEFINED are not considered valid."

So no attempt is made to coordinate display units with the unit type that generated the FormatOptions.

Note that SetFormatOptions does throw an exception for invalid combinations of UnitType and DisplayUnitType if you add this code to the check:

```csharp
oFormatOption.DisplayUnits = eDisplayUnitType;
using (Transaction t = new Transaction(doc))
{
t.Start("Set options");
units.SetFormatOptions(unitType, oFormatOption);
t.Commit();
}
```

On the other hand, there really is an easier way to achieve this:

Call UnitUtils.GetValidDisplayUnits(UnitType), which directly returns valid types without any further complications.

#### Self-contained RevitPythonShell Deployment

Daren Thomas points out the
[new RevitPythonShell deployment functionality](http://darenatwork.blogspot.ch/2013/05/deploying-rps-scripts-with.html) that
enables the creation of a self-contained set of DLLs for deploying RevitPythonShell scripts without requiring the user to install RevitPythonShell.
You still need to create an installer, and a HelloWorld example with a sample InnoSetup installer is provided.