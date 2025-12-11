---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.7
content_type: code_example
optimization_date: '2025-12-11T11:44:14.377548'
original_url: https://thebuildingcoder.typepad.com/blog/0675_set_family_param.html
post_number: '0675'
reading_time_minutes: 2
series: family
slug: set_family_param
source_file: 0675_set_family_param.htm
tags:
- csharp
- elements
- family
- geometry
- parameters
- python
- revit-api
title: Set Family Parameter Requires Type
word_count: 392
---

### Set Family Parameter Requires Type

I was running down to town to have tea with my friend Otto on Saturday morning, and happened to see this autumn leaf lying on the ground.
Turned back, picked it up, and made a picture of it in the sunshine on Otto's veranda.
It is completely unretouched, I promise!
![Autumn leaf](file:////j/photo/jeremy/2011/2011-11-12_autumn_leaf/autumn_leaf_narrow.jpg)

Back to the Revit API, here is a short note on another issue run into by Ishwar Nagwani, Technical Consultant in our Bangalore organisation, who recently contributed the
[pick points to create a new a floor](http://thebuildingcoder.typepad.com/blog/2011/11/pick-corners-and-create-floor.html) sample, and answered by Harry Mattison of the Revit development team.

**Question:** I am trying to update an existing family parameter and set its value, but it is always throwing an InvalidOperationException saying "There is no current type".
What is wrong?

Here is some minimal sample code to reproduce the issue.
Only the family file name is hardcoded.
```python
public class Command : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;

    uiapp.OpenAndActivateDocument(
      "C:\\Projects\\FY12\\GE Revit\\SWBD-AV2.rfa" );

    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    FamilyManager familyMgr = doc.FamilyManager;

    FamilyParameter param = familyMgr.get\_Parameter(
      "Width" );

    if( null == param )
    {
      param = doc.FamilyManager.AddParameter(
        "Width", BuiltInParameterGroup.PG\_GEOMETRY,
        ParameterType.Length, true );
    }

    familyMgr.Set( param, 0.2 ); // set the value

    return Result.Succeeded;
  }
}
```

**Answer:** The exception message actually says it all, albeit rather succinctly: "There is no current type".

FamilyManager.Set is used to set the value of a parameter for the current family type.
The family you are working with had no types defined, therefore Set could not work.
You can remedy this by adding a family type manually or with the following two lines:
```csharp
  if( doc.FamilyManager.Types.Size == 0 )
    doc.FamilyManager.NewType( "Type 1" );
  familyMgr.Set( param, 0.2 );
```

Again, thanks to Ishwar for sharing this!

#### Addendum – SetValueString Works Without Type

Alexander, Александр Пекшев, points out another possibility below, saying:

You do not have to create a type! You can use the method `SetValueString` instead of the `Set` method – it does not require pre-creation of a type, and does what it needs – it sets the value for the parameter!