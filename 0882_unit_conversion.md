---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: news
optimization_date: '2025-12-11T11:44:14.790504'
original_url: https://thebuildingcoder.typepad.com/blog/0882_unit_conversion.html
post_number: 0882
reading_time_minutes: 2
series: general
slug: unit_conversion
source_file: 0882_unit_conversion.htm
tags:
- csharp
- family
- parameters
- revit-api
title: Unit Conversion Utility Update
word_count: 344
---

### Unit Conversion Utility Update

Here is a Revit 2013 update of the Revit 2011
[unit conversion utility](http://thebuildingcoder.typepad.com/blog/2011/01/unit-conversion-and-new-blogs.html),
originally posted and
[described for Revit 2010](http://thebuildingcoder.typepad.com/blog/2009/08/unit-conversion.html), by
[Harry Mattison](http://boostyourbim.wordpress.com).

In his own words:

I was looking at the great
[unit conversion utility](http://thebuildingcoder.typepad.com/blog/2011/01/unit-conversion-and-new-blogs.html) and
created a
[new version upgraded to use the Revit 2013 API](zip/UnitConversions2013.zip)
that I thought you readers might enjoy.

I ran into one issue that required a code enhancement to catch an exception thrown by many parameters, such as HVAC\_Temperature, HVAC\_Slope, etc.

The rest of the code handles error conditions nicely, so I was surprised to see these uncaught exceptions.

I solved it by adding a try/catch statement around the existing call to AddParameter in frmUnitConversions.cs:

```csharp
  if( null == \_Parameter )
  {
    // Create a new parameter

    try
    {
      \_Parameter
        = \_DbDocument.FamilyManager.AddParameter(
          parameterTypeToCreate.ToString(),
          BuiltInParameterGroup.INVALID,
          parameterTypeToCreate,
          false );
    }
    catch( Exception ex )
    {
      TaskDialog.Show( "Error",
        "Cannot create parameter '"
        + parameterTypeToCreate.ToString()
        + "' with parameter type '"
        + parameterTypeToCreate
        + "' because of exception\n"
        + ex.Message.ToString() );

      return;
    }
  }
```

I think I figured out what is causing this and thought it would be interesting to share with your blog readers:

I am creating family parameters from a shared parameter file.
Some parameters have types that are not supported in the version of Revit that I am currently using, which is Revit Architecture.

I created this function to detect the situation:
```csharp
  public static bool canGetFormatOptions(
    Document doc,
    FamilyParameter familyParameter )
  {
    try
    {
      UnitType parameterUnitType
        = ConvertParameterTypeToUnitType(
          familyParameter.Definition.ParameterType );

      doc.ProjectUnit.get\_FormatOptions(
        parameterUnitType );

      return true;
    }
    catch
    {
      return false;
    }
  }
```

I use it like this:

```csharp
  if( !canGetFormatOptions(
    famDoc, familyParameter ) )
  {
    TaskDialog.Show( "Error",
      "Cannot set parameter value.\nUnit Type "
      + ConvertParameterTypeToUnitType(
        familyParameter.Definition.ParameterType )
      + " is not supported in Revit "
      + famDoc.Application.Product.ToString() );
  }
```

Take care.

The updated code is provided above.

Many thanks to Harry for this update with his enhancement and analysis!