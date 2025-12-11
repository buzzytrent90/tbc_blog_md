---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: qa
optimization_date: '2025-12-11T11:44:13.583256'
original_url: https://thebuildingcoder.typepad.com/blog/0226_unit_suffix.html
post_number: '0226'
reading_time_minutes: 6
series: general
slug: unit_suffix
source_file: 0226_unit_suffix.htm
tags:
- parameters
- revit-api
title: Unit Suffix and the ProjectUnit SDK Sample
word_count: 1204
---

### Unit Suffix and the ProjectUnit SDK Sample

We delved into various aspects of units in several previous posts:

- [Basics on units](http://thebuildingcoder.typepad.com/blog/2008/09/units.html).- [Formatting unit strings](http://thebuildingcoder.typepad.com/blog/2008/11/formatting-units-strings.html).- [Unit types and format options](http://thebuildingcoder.typepad.com/blog/2009/01/unit-types-and-format-options.html).- [Unit conversion](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html).

And still the topic is not exhausted. Here is a new point to consider:

**Question:** Is there a way to retrieve the unit for a parameter?
I can read a parameter's value and its Display Unit Type but not the unit's symbol, the display string suffix used to represent the unit in the user interface.
For example, for a length parameter I would like to retrieve FT or m for the unit symbol, or SF for an area parameter.
I can read the parameter value and unit suffix combined by using AsValueString, but what about retrieving the unit symbol separately.
If it is possible to get the parameter unit symbol, could you send me a short code snippet about it?

If the Revit API does not provide parameter value unit, then what is the use of FormatOption.Unitsymbol?
For example, UST\_FT\_SUP\_2? Can you give me an application of UST symbols?

Also, one way I can think of putting a unit next to a parameter value in the user interface is to write a long switch
statement and determine the name of the parameter and its DUT.
For example, if the parameter name is Area and its DUT is DUT\_SQUARE\_FEET then unit is SF, etc.
Am I right?

**Answer:** Let us dive into the unit handling and the classes and enumerations used for this a little bit deeper before returning to the question.

The unit suffix you mention is a user defined setting, so it is actually not directly defined by the parameter itself.
The parameter is
defined as having a certain unit type such as length or area, and it is completely up to the user to define how values of
this type are represented in the user interface.

The UnitType enumeration lists 81 different available unit types, the simplest of which are UT\_Length and UT\_Area.

The unit types are further grouped by discipline into unit groups, represented by the UnitGroup enumeration with the
following members:

- UG\_Common- UG\_Electrical- UG\_HVAC- UG\_Piping- UG\_Structural

Length and area belong to the common group, whereas more specific unit types such as force or moment are classified as
according to discipline, e.g., structural.

How each unit is represented in the user interface is defined by the ProjectUnit and FormatOptions classes.

The description of the ProjectUnit class in the Revit API help file includes an example of accessing and listing this
information.

The FormatOptions class contains the unit information of the current user settings in the project.
It members include

- Group: the unit's type group.- Name: the unit's type name.- Rounding: the unit's type rounding and accuracy.- Units: the unit type itself.- Unitsymbol: the unit's type symbol.

We mentioned this class once in the past in the following blog post:
http://thebuildingcoder.typepad.com/blog/2009/01/unit-types-and-format-options.html

A full example of managing the user-defined unit settings is provided by the SDK sample ProjectUnit. It demonstrates how to
read and set units and format options. It defines an external command which displays a dialogue box listing all the units in
the current project and displaying their format information, including the decimal separator and slope type.
![Revit SDK sample ProjectUnit](img/project_unit.png)

This functionality includes

- List all units in current project and display each unit's format information.- Display the current project units decimal symbol type of set it to comma or dot.- Display the current project units slope type and set it to Rise or Angle.

The units listing uses the unit group classification by discipline, so a list all the available unit groups or disciplines is
displayed and used to classify the units. When one unit group is selected in the list box, the corresponding project units
and their name, format information, unit suffix type, and rounding is displayed.

- The name is the user interface display string such as Length or Area.- The format information defines the display unit type such as DUT\_KIP\_FEET\_PER\_DEGREE\_PER\_FOOT, represented by the
    DisplayUnitType enumeration. This basically represents a numerical conversion factor from the Revit internal database units
    to the number you see in the user interface units.- The unit suffix type specifies a string suffix appended to the value to communicate what units are being displayed.
      Possible values are defined by the UnitSymbolType enumeration, such as UST\_NONE for length or UST\_M\_SUP\_2 for area.- The rounding or accuracy is a number like 1 or 0.2.

The unit's symbol that you mention in your question might correspond either to the format information or the unit suffix.
There is a considerable overlap between these two enumerations.

The parameter definition has a ParameterType property, which returns the user visible interpretation of the parameter data.
The Revit API help file provides a short code example of using this property.
It returns a ParameterType enumeration value, and many of these values are associated with a
unit type.
The parameter definition also has a ParameterGroup property returning the group ID of the parameter definition, which is
represented by the BuiltInParameterGroup enumeration.
That can also give a hint on what units are appropriate.

To return to your original question,
yes, there is a property on the parameter class itself for the DisplayUnitType as well as its definition.
So you can simply use Parameter.DisplayUnitType to determine the DUT, just as you say.
Yes, I do not see any access in the API to retrieve the display string of a DUT.
These strings might be language dependent, and no API method has been provided to retrieve them for the localised product data.
You will of course obtain the appropriate unit suffix by calling AsValueString.

For an application of the UnitSymbolType enumeration values, I can imagine that they might in fact be useful for what you
are trying to achieve, since each enumeration name does actually describe the unit suffix. For instance:

- UST\_M represents m.- UST\_M\_CARET\_2 represents m^2.- UST\_M\_SUP\_2 represents m2.

So you could possibly reduce your switch statement to a shorter algorithm putting together your own unit suffix string based on
interpreting the names of these enumeration values.

Alternatively, you could write a full switch statement such as you describe. It could be based on either the DUT or on the
UST, since there is significant overlap between the two.
This depends on what you actually want to display in the list.
As you can see above, the default setting on my system for a length is DUT\_Millimetres and UST\_None, which are different, so depending on which of the two I base the unit symbol display on, I would get a different result.