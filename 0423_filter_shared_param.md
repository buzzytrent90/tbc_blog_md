---
post_number: "0423"
title: "ElementParameterFilter with a Shared Parameter"
slug: "filter_shared_param"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'parameters', 'python', 'revit-api', 'rooms', 'selection', 'walls']
source_file: "0423_filter_shared_param.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0423_filter_shared_param.html"
---

### ElementParameterFilter with a Shared Parameter

I thought I had already covered everything I could on filtered element collectors, but far failed.
Here is a pretty obvious question that just came up on using an ElementParameterFilter with a shared parameter:

**Question:** I have successfully used element parameter filter to find elements with a particular parameter value, using 'built-in' parameters, such as BuiltInParameter.ELEM\_ROOM\_NUMBER.

What I want to do now, is to use a similar filter to find all elements that have a value for a shared parameter. For example:

I have a shared parameter called "ROOM\_LINK" and I want to find any elements in the model that have a value for ROOM\_LINK.

In this case, ROOM\_LINK is applicable to Rooms and Generic Models.

If it is necessary or helpful, this could be done with a non-shared Project parameter, but I think using a shared parameter is more appropriate.

Do you have a little sample of code to show how to use a shared parameter (or other non-built-in parameter) in a filter?

**Answer:** Congratulations on your successful use of the element parameter filter.

As you probably noticed, I already published a couple of discussions of related topics, and two of these are specifically on
[parameter filters](http://thebuildingcoder.typepad.com/blog/2010/06/parameter-filter.html) with an appended
[little correction](http://thebuildingcoder.typepad.com/blog/2010/06/element-name-parameter-filter-correction.html).
The parameter filter post includes a list of pointers to other filtered element collector examples.

When using a parameter filter, you have to set up the ParameterValueProvider.

Its constructor always takes an element id as an argument.

In the samples I discussed so far, this element id argument was defined by converting a built-in parameter enumeration value.

The element id can however also be the real element id of any given real parameter, which can be retrieved by any of the normal means, e.g. by display name like in the following code:
```python
public class ParamFilterTest : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    Wall wall = uidoc.Selection.PickObject(
      Autodesk.Revit.UI.Selection.ObjectType.Element )
      .Element as Wall;

    Parameter parameter = wall.get\_Parameter(
      "Unconnected Height" );

    ParameterValueProvider pvp
      = new ParameterValueProvider( parameter.Id );

    FilterNumericRuleEvaluator fnrv
      = new FilterNumericGreater();

    FilterRule fRule
      = new FilterDoubleRule( pvp, fnrv, 20, 1E-6 );

    ElementParameterFilter filter
      = new ElementParameterFilter( fRule );

    FilteredElementCollector collector
      = new FilteredElementCollector( doc );

    // Find walls with unconnected height
    // less than or equal to 20:

    ElementParameterFilter lessOrEqualFilter
      = new ElementParameterFilter( fRule, true );

    IList<Element> lessOrEqualFounds
      = collector.WherePasses( lessOrEqualFilter )
        .OfCategory( BuiltInCategory.OST\_Walls )
        .OfClass( typeof( Wall ) )
        .ToElements();

    TaskDialog.Show( "Revit", "Walls found: "
      + lessOrEqualFounds.Count );

    return Result.Succeeded;
  }
}
```

Retrieving the parameter by display name should work like this for a shared parameter as well.