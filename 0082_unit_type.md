---
post_number: "0082"
title: "Unit Types and Format Options"
slug: "unit_type"
author: "Jeremy Tammik"
tags: ['revit-api', 'sheets', 'views']
source_file: "0082_unit_type.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0082_unit_type.html"
---

### Unit Types and Format Options

Here is a question from
[Matt Mason](http://cadappdev.blogspot.com)
and some exhaustive research results by Saikat Bhattacharya of Autodesk on unit types and their format options.

To provide some context, UnitType is an enumeration listing all the unit types supported by Revit. The FormatOptions class is used to manage information for the ProjectUnit.FormatOptions property, which returns a FormatOptions instance containing the user's current project unit settings for each UnitType queried. Through the project units and the format options management, each unit type is associated with one FormatOptions instance, which provides the following properties and values:

- Group, a UnitGroup enumeration value,
- Name, a string,
- Units, a display unit type, represented by a DisplayUnitType enumeration value,
- Symbol, a UnitSymbolType enumeration value,
- Rounding, a double value.

In this context, Matt asks:

**Question:**
When attempting to get the FormatOptions for a given UnitType, I am getting exceptions for certain UnitTypes:

- AreaForceScale
- DecSheetLength
- Electrical Apparent Power
- Electrical Current
- Electrical Frequency
- Electrical Luminance
- Electrical Potential
- Electrical Power
- Electrical Power Density
- ForceScale
- HVAC \*
- Linear Force Scale
- Moment Scale
- Number
- Pipe Size
- Piping \*
- SheetLength
- SiteAngle
- WireSize

Some of these might be related to the fact that I am running the Revit Architecture flavour, but it is not clear to me that all of these are based on Revit flavour issues.
Are some of these legacy or otherwise invalid types?

**Answer:**
Saikat created the following mapping of the list of unit types provided with the disciplines in RAC as well as RME.
Most of the unit types listed below belong to the MEP flavour of Revit and thus do not throw an exception in RME:

- AreaForceScale - Structural
- DecSheetLength -
- Electrical Apparent Power - Electrical
- Electrical Current - Electrical
- Electrical Frequency - Electrical
- Electrical Luminance - Electrical
- Electrical Potential - Electrical
- Electrical Power - Electrical
- Electrical Power Density - Electrical
- ForceScale - Structural
- HVAC \* - HVAC
- Linear Force Scale - Structural
- Moment Scale - Structural
- Number -
- Pipe Size - Piping
- Piping \* - Piping
- SheetLength -
- SiteAngle -
- WireSize - Electrical

After contacting the engineering team for more details, he compiled a more comprehensive list including 80 unit types with their format option groups and names, and additional notes on some of them.
Here is a spreadsheet
[ProjectUnit.csv](zip/ProjectUnit.csv)
with the complete mapping in a neutral comma delimited CSV format.

After reviewing these results, Matt adds a note that the four 'special' unit types DecSheetLength, SheetLength, Number, and SiteAngle are 'fixed' and will throw an exception if you attempt to access them in any Revit Flavour.
Saikat adds: As mentioned in the CSV file, there are more than the four special cases that Matt refers to as 'special'.
The CSV file contains others which apply to all disciplines and also cannot be edited, which Matt refers to as 'fixed', and includes some notes on each of them.