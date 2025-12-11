---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.4
content_type: documentation
optimization_date: '2025-12-11T11:44:14.104190'
original_url: https://thebuildingcoder.typepad.com/blog/0525_fam_inst_missing_level.html
post_number: '0525'
reading_time_minutes: 2
series: general
slug: fam_inst_missing_level
source_file: 0525_fam_inst_missing_level.htm
tags:
- csharp
- elements
- family
- filtering
- levels
- parameters
- revit-api
- windows
title: Family Instance Missing Level Property
word_count: 452
---

### Family Instance Missing Level Property

Here is another contribution by Rudolf Honke of
[acadGraph CADstudio GmbH](http://www.acadgraph.de).
He says:

"And now for something completely different" ... after all the UI Automation topics:

I often need to collect all items on a specific level, for instance window family instances, and use a method like the following to do so:
```csharp
List<Element> GetWindowsByLevel(
  Document doc,
  Level level )
{
  List<Element> elementList = new List<Element>();

  FilteredElementCollector collector
    = new FilteredElementCollector( doc )
      .OfCategory( BuiltInCategory.OST\_Windows );

  ParameterValueProvider provider
    = new ParameterValueProvider(
      new ElementId(
        BuiltInParameter.FAMILY\_LEVEL\_PARAM ) );

  FilterNumericRuleEvaluator evaluator
    = new FilterNumericEquals();

  ElementId idRuleValue = level.Id;

  FilterElementIdRule rule
    = new FilterElementIdRule(
      provider, evaluator, idRuleValue );

  ElementParameterFilter filter
    = new ElementParameterFilter( rule );

  elementList.AddRange(
    collector.WherePasses( filter ).ToElements() );

  return elementList;
}
```

Strangely enough, it may occur that some elements are not found by this method, because their level property has not been correctly set.

Here are the element properties of a window with its level property (Ebene in German) properly set:

![Window with Level property properly set](img/rh_level_ok.png)

Other window instances, however, may be lacking this property:

![Window with Level property missing](img/rh_level_missing.png)

If we take a look at the second window instance parameters using the RevitLookup 'snoop built-in enums' functionality, we see that it has no FAMILY\_LEVEL\_PARAM parameter:

![Window lacking FAMILY_LEVEL_PARAM parameter](img/rh_level_missing_bips.png)

This window instance also has a null value for its Level **property**:

![Window has null Level property](img/rh_level_missing_property.png)

Sometimes, you can set a parameter to affect a property.
Similarly, you can move or rotate an element to change its location point or curve.

Sometimes a property and the parameter that should relate to this property are inconsistent with each other, but that's another point.

In the element property page, you cannot set the property, because it is not displayed there at all.

Therefore, this is one of the rare occasions where you can achieve something via the API that cannot be done through the user interface.

In this case, you can use this little workaround:

- Retrieve all the levels, e.g. sorted by elevation or by name.- Display their names in a dialog and provide an opportunity to the user to select the desired one:

![Level selector](img/rh_level_select.png)

- Set the built-in parameter value of the family instance lacking the level property to the selected level element id:

```csharp
  windowWithoutLevelParam.get\_Parameter(
    BuiltInParameter.FAMILY\_LEVEL\_PARAM )
    .Set( levels[levelDialogSelectedIndex].Id );
```

Once assigned programmatically, the level property also becomes visible to the user:

![Window Level property displayed](img/rh_level_property_displayed.png)

After this adjustment, the GetWindowsByLevel method returns the results it is designed for as expected.

Many thanks to Rudolf for this detailed analysis and workaround!