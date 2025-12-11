---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.3
content_type: qa
optimization_date: '2025-12-11T11:44:14.085908'
original_url: https://thebuildingcoder.typepad.com/blog/0515_fam_param_guid.html
post_number: '0515'
reading_time_minutes: 3
series: general
slug: fam_param_guid
source_file: 0515_fam_param_guid.htm
tags:
- csharp
- elements
- family
- parameters
- revit-api
title: Access to Family Parameter GUID
word_count: 555
---

### Access to Family Parameter GUID

Here is a question that came up several times.
It also demonstrates a neat (if somewhat time consuming) use of Reflection to access a property that otherwise would be inaccessible.
Here is the original question:

**Question:** How can I reliably tell whether a parameter on a family document is
a shared parameter, or is actually a family parameter?
If it is a shared parameter, how can I retrieve its original GUID as it was when it was originally added to the family?

Note that I need the **original** GUID used to load that parameter, and cannot rely on the current shared parameters file, which may not be the same one as originally used.

I tried to solve this directly, but the retrieving the parameter definition of a parameter on a family only reports it as being an InternalDefinition, even if it is actually a shared parameter, which should return an ExternalDefinition instead.
I tested it running a piece of code like this:
```csharp
  FamilyParameter parameter
    = <get a family parameter
        ref from the FamilyManager>

  ExternalDefinition externalDef
    = parameter.Definition as ExternalDefinition;

  InternalDefinition internalDef
    = parameter.Definition as InternalDefinition;
```

The externalDef variable always ends up being null, and the internalDef one has a valid value.

**Answer:** This was resolved for non-family parameters by the Revit 2011 API.
The Revit API help file RevitAPI.chm lists the following feature in the 'What's New' section:

#### Extract GUID from a Parameter

The new properties

- Parameter.IsShared- Parameter.GUID

identify if a given parameter is a shared parameter, and if it is, extract its GUID.

Unfortunately, this only works for standard parameters, and not for family parameters.

Using the debugger on an object of type Autodesk.Revit.DB.FamilyParameter, one can see
something called "m\_Parameter" which **does** have the properties IsShared and GUID which **do** have the correct values.

Unfortunately, again, you cannot simply cast an Autodesk.Revit.DB.FamilyParameter to an Autodesk.Revit.DB.Parameter type.

Happily, it is possible to access these properties using .NET System.Reflection.

Here is a workaround using this to access these properties which works in both Revit 2010 and 2011.
It retrieves the underlying family parameter definition IsShared and GUID properties and returns true if the family parameter is shared and has a GUID:
```csharp
bool GetFamilyParamGuid(
  FamilyParameter fp,
  out string guid )
{
  guid = string.Empty;

  bool isShared = false;

  System.Reflection.FieldInfo fi
    = fp.GetType().GetField( "m\_Parameter",
      System.Reflection.BindingFlags.Instance
      | System.Reflection.BindingFlags.NonPublic );

  if( null != fi )
  {
    Parameter p = fi.GetValue( fp ) as Parameter;

    isShared = p.IsShared;

    if( isShared && null != p.GUID )
    {
      guid = p.GUID.ToString();
    }
  }
  return isShared;
}
```

I implemented a new external command CmdFamilyParamGuid in The Building Coder samples to test this method:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication app = commandData.Application;
  Document doc = app.ActiveUIDocument.Document;

  if( !doc.IsFamilyDocument )
  {
    message =
      "Please run this command in a family document.";

    return Result.Failed;
  }
  else
  {
    bool isShared;
    string guid;

    FamilyManager mgr = doc.FamilyManager;

    foreach( FamilyParameter fp in mgr.Parameters )
    {
      isShared = GetFamilyParamGuid( fp, out guid );
    }
    return Result.Succeeded;
  }
}
```

To run and test this, just open any family document and include at least one shared parameter.

Here is
[version 2011.0.82.0](zip/bc_11_82.zip)
of The Building Coder samples including the complete source code and Visual Studio solution with the new command.