---
post_number: "0200"
title: "Electrical Settings and Lighting Fixtures"
slug: "electrical_and_lighting"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'revit-api', 'selection']
source_file: "0200_electrical_and_lighting.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0200_electrical_and_lighting.html"
---

### Electrical Settings and Lighting Fixtures

Here are two recent questions on
[electrical settings of distribution systems](#1)
and
[family instances for lighting fixtures](#2)
handled by Saikat Bhattacharya and Joe Ye, respectively.

By the way, this is a special occasion, namely the 200th post to The Building Coder.
I will resist the temptation to declare this as a birthday post like I did
[post number 100](http://thebuildingcoder.typepad.com/blog/2009/02/utilizing-revit-api-resources.html),
because the first real annual birthday of The Building Coder is coming up quite soon now anyway, and we don't want to exaggerate, do we?

#### Reading and Writing the Electrical Settings of Distribution Systems

**Question:**
Is there any way to use the Revit API to read and write the data under Electrical Settings, e.g. the Distribution System rows?

**Answer:**
To do a quick check, I used the RvtMgdDbg tool and found out that the active document contains a DistributionSysType container which lists all the distribution systems available from the user interface.
Here are the electrical settings displayed in the user interface:

![Electrical Settings](img/electrical_settings.jpg)

Here, we have selected the DistributionSysType instance using
RvtMgdDbg
to show the same information as the user interface as accessed through the API:

![DistributionSysType in RvtMgdDbg](img/electrical_settings2.jpg)

Thus, the DistributionSysType class might be what you looking for.

#### Creating Hosted Family Instances for Lighting Fixtures

**Question:**
I am loading a batch of lighting fixtures using .NET but I am having some issues.
I programmatically set the host to be the ceiling by using parameters showing that the host is the ceiling when the fixture is loaded.
However, the properties of the resulting fixture do not reflect this.
Also, the fixture created cannot be copied.
In sum, the properties of the fixtures are not the same as the properties of a fixture created manually and inserted by Revit itself.
I used the following code to load the fixture:

```
FamilyInstance inst
  = doc.Create.NewFamilyInstance(
    location, symbol, ceil,
    StructuralType.NonStructural );
```

In this call, location is the location of the fixture.
I set its Z coordinate to zero in order to inherit the host elevation, but this does not work.
Symbol is the family symbol to be inserted, and ceil the selected host ceiling.

**Answer:**
I reproduced the issue.
Using your method, I found the created light fixture is located above the ceiling, although the Z elevation is the same.
The host property is null, as is the parameter value of HOST\_ID\_PARAM.
So I think the issue is caused by the level setting.

I changed the code to use another override of NewFamilyInstance, which takes the level as an argument:

```
Document.NewFamilyInstance(
  XYZ,
  FamilySymbol,
  Element,
  Level,
  StructuralType )
```

The lighting fixture now has a valid host property. Here is the updated code for the Execute method of the external command:

```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

Autodesk.Revit.Creation.Filter cf
  = app.Create.Filter;

// get a lighting fixture family symbol:

Filter f1 = cf.NewTypeFilter(
  typeof( FamilySymbol ) );

Filter f2 = cf.NewCategoryFilter(
  BuiltInCategory.OST\_LightingFixtures );

Filter f = cf.NewLogicAndFilter( f1, f2 );

List<Element> symbols = new List<Element>();
doc.get\_Elements( f, symbols );

FamilySymbol sym = null;

foreach( Element elem in symbols )
{
  sym = elem as FamilySymbol; // get the first one.
  break;
}

if( null == sym )
{
  message = "No lighting fixture symbol found.";
  return CmdResult.Failed;
}

// pick the ceiling:

doc.Selection.StatusbarTip
  = "Please select ceiling to host lighting fixture";

doc.Selection.PickOne();

Element ceiling = null;

foreach( Element elem in doc.Selection.Elements )
{
  ceiling = elem as Element;
  break;
}

// get the level 1:

ElementIterator it = doc.get\_Elements(
  typeof( Level ) );

Level level = null;

while( it.MoveNext() )
{
  level = it.Current as Level;

  if( level.Name.Equals( "Level 1" ) )
    break;
}

if( null == level )
{
  message = "Level 1 not found.";
  return CmdResult.Failed;
}

// create the family instance:

XYZ p = app.Create.NewXYZ( -43, 28, 0 );

FamilyInstance instLight
  = doc.Create.NewFamilyInstance(
    p, sym, ceiling, level,
    StructuralType.NonStructural );

return CmdResult.Succeeded;
```

Thank you very much Joe and Saikat for these answers!