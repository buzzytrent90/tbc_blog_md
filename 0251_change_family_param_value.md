---
post_number: "0251"
title: "Change Family Parameter Value"
slug: "change_family_param_value"
author: "Jeremy Tammik"
tags: ['csharp', 'family', 'geometry', 'parameters', 'revit-api']
source_file: "0251_change_family_param_value.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0251_change_family_param_value.html"
---

### Change Family Parameter Value

Here is another issue dealing with family parameters, after the notes on how to
[read the values of family parameters](http://thebuildingcoder.typepad.com/blog/2009/11/family-parameter-value.html) and
looking at the family parameter values via the
[part atom export](http://thebuildingcoder.typepad.com/blog/2009/11/extract-part-atoms.html).

**Question:** I'm using the AutoParameter SDK example to add parameters to my families.
It was customized so that only specific parameters from the shared parameters file are added to the families (i.e. we have a large shared parameters file, and we only want certain ones added to these families) using a switch statement along with the parameter names to find.
When we add a parameter with a checkbox, it's automatically checked.
We need to go back after it was added and uncheck it.
I went through a number of SDK examples but haven't found one that helps.
Can you send me a snippet that shows how to do this?
I am using C#.

**Answer:** Do the parameters you have added in the family appear checked in the user interface of the project after being loaded into the document environment?
In that case, I assume that the parameters you have added have a Boolean data type and an initial value of True.
If so, the check mark should be removed if you set their initial value to False.
For instance, in the
[RFA family labs](http://thebuildingcoder.typepad.com/blog/2009/10/revit-family-creation-api-labs.html),
the addParameter method adds two real-values parameters:
```csharp
void addParameters()
{
  FamilyManager mgr = \_rvtDoc.FamilyManager;

  // API parameter group for Dimension is PG\_GEOMETRY:
  //
  FamilyParameter paramTw = mgr.AddParameter(
    "Tw", BuiltInParameterGroup.PG\_GEOMETRY,
    ParameterType.Length, false );

  FamilyParameter paramTd = mgr.AddParameter(
    "Td", BuiltInParameterGroup.PG\_GEOMETRY,
    ParameterType.Length, false );

  // set initial values:
  //
  double tw = mmToFeet( 150.0 );
  double td = mmToFeet( 150.0 );
  mgr.Set( paramTw, tw );
  mgr.Set( paramTd, td );
}
```

In your case, you might be able to use something like:
```csharp
  FamilyParameter paramDan = mgr.AddParameter(
    "Dan", BuiltInParameterGroup.PG\_TEXT,
    ParameterType.YesNo, true );

  mgr.Set( paramDan, 0 );
```

**Response:** Thanks! that's exactly what I needed.