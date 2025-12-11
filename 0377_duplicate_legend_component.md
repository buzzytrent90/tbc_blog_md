---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: code_example
optimization_date: '2025-12-11T11:44:13.843806'
original_url: https://thebuildingcoder.typepad.com/blog/0377_duplicate_legend_component.html
post_number: '0377'
reading_time_minutes: 2
series: general
slug: duplicate_legend_component
source_file: 0377_duplicate_legend_component.htm
tags:
- elements
- family
- parameters
- python
- revit-api
- selection
- transactions
title: Duplicate Legend Component
word_count: 465
---

### Duplicate Legend Component

A number of Revit elements can still not be created programmatically, because the API does not provide the appropriate creation methods for them.

Joe Ye just handled a case asking how to create a new legend component, and Harry Mattison suggested an exciting new possibility that we were previously not aware of.

This method allows us to duplicate many kinds of elements with an existing instance inserted. Here is the original question as a starting point:

**Question:** Is it possible to add a new Legend Component to the Legend programmatically?
I have found no API for the Legend Component.
Is there some workaround – for example invoking a keyboard shortcut for it?

Is it possible to copy an existing Legend Component and change the family it is referring to?

**Answer:** The Revit API does not currently expose any methods that allow us to add a new legend component.

The good news is that there is a workaround to create a new element by copying an existing one.

The solution is to put the element that you would like to copy into a group, place an instance of the group in the document, and then ungroup the two groups.

The result is that the existing element remains and a new element is created in addition.

Here is a new Building Coder sample command CmdDuplicateElements which demonstrates this, and also provides a first example of a command modifying the Revit model using manual transaction mode, which requires it to start and commit its own transaction:
```python
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
class CmdDuplicateElements : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    UIDocument uidoc = app.ActiveUIDocument;
    Document doc = uidoc.Document;

    Transaction trans = new Transaction( doc,
      "Duplicate Elements" );

    trans.Start();

    Group group = doc.Create.NewGroup(
      uidoc.Selection.Elements );

    LocationPoint location = group.Location
      as LocationPoint;

    XYZ p = location.Point;
    XYZ newPoint = new XYZ( p.X, p.Y + 10, p.Z );

    Group newGroup = doc.Create.PlaceGroup(
      newPoint, group.GroupType );

    group.Ungroup();

    ElementSet eSet = newGroup.Ungroup();

    // change the property or parameter values
    // of elements in eSet as required...

    trans.Commit();

    return Result.Succeeded;
  }
}
```

To test this command, you can select a legend component and launch the command.
A new legend component will be created.
You can then change the new legend's parameter
Component Type to change the legend type.

I created three legend components and three other detail objects, selected them, and lauched the command, which produced the six copied elements ten feet higher up like this:

![Duplicated elements](img/duplicated_elements.png)

Here is
[version 2011.0.70.0](zip/bc_11_70.zip)
of The Building Coder sample source code and Visual Studio solution including the new command.

Many thanks to Joe for handling the case and Harry for suggesting this approach.