---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.581897'
original_url: https://thebuildingcoder.typepad.com/blog/0225_enumerations.html
post_number: '0225'
reading_time_minutes: 3
series: general
slug: enumerations
source_file: 0225_enumerations.htm
tags:
- csharp
- parameters
- revit-api
- walls
- windows
title: Display Strings and Enumerations
word_count: 514
---

### Display Strings and Enumerations

**Question:** I need to get the string value displayed in the Revit user interface for some parameters whose values only contain an integer value, such as:

- Wall Function: the API returns an integer value, whereas the UI displays "Exterior".- Wrapping at Inserts: again, the API returns an integer value, whereas the UI displays "Both".

Is there a general way to get a mapping between the integer values and the display strings for all such items in Revit?

**Answer:** For a parameter containing a double value representing a length or an area, you can retrieve the raw internal database value using the Parameter.AsDouble method, and the AsValueString method will return the string representation as displayed in the Revit user interface, honouring the current unit settings.

Unfortunately, for the enumerated parameter values you are interested in, the parameter value is just a normal integer value, and AsValueString can do nothing to convert it to the user interface display string.
Please note that these display strings are localised, i.e. language dependent, so they cannot simply be hard-wired into the API.
Revit does not provide any API method to retrieve the mapping from the internal integer values to the display strings programmatically.
The mapping can be created manually by setting every option of the parameter in the user interface and then checking its integer value using RvtMgdDbg, which is obviously tedious and inefficient.

In the case of the wall function, this results in the following value mapping:

- 0 Interior- 1 Exterior- 2 Foundation- 3 Retaining- 4 Soffit- 5 Core-shaft

The wall function is also represented by the WallFunction Enumeration in the API, which has the following values:

- Soffit- Retaining- Foundation- Exterior- Interior

Converting these enumeration values to integers does return the same numbers as those listed above.
However, in this case, the Core-shaft entry is missing in the enumeration.

Whenever you run into an integer-valued parameter like this, it is worth while checking the Revit API help file to see whether a corresponding enumeration type is available.
You may of course run into parameters for which this is not the case, or where some of the values displayed in the user interface are missing in the API, but in many cases this can save at least some of the manual effort described above.

My colleague Augusto Goncalves points out that this is an apt place to mention that you can use the .NET Enum class to get some information about the enumeration values, for example like this:
```csharp
// list enum values:

string[] enumValues = Enum.GetNames(
  typeof( Autodesk.Revit.Enums.WallFunction ) );

StringBuilder values
  = new StringBuilder( "Values are: " );

foreach( string s in enumValues )
{
  values.AppendFormat( "\n{0}", s );
}

System.Windows.Forms.MessageBox.Show(
  values.ToString() );

// convert an enum value to string:

Autodesk.Revit.Enums.WallFunction aValue
  = Autodesk.Revit.Enums.WallFunction.Interior;

string aValueName = Enum.GetName(
  aValue.GetType(), aValue );

System.Windows.Forms.MessageBox.Show(
  String.Format( "The value is: {0}", aValueName ) );
```

By the way, conversion to a string can also be achieved even more simply and directly by using aValue.ToString().