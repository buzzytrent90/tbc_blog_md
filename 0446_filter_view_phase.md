---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.1
content_type: code_example
optimization_date: '2025-12-11T11:44:13.959075'
original_url: https://thebuildingcoder.typepad.com/blog/0446_filter_view_phase.html
post_number: '0446'
reading_time_minutes: 2
series: filtering
slug: filter_view_phase
source_file: 0446_filter_view_phase.htm
tags:
- elements
- filtering
- parameters
- python
- revit-api
- transactions
- views
title: Filter for View and Phase
word_count: 308
---

### Filter for View and Phase

Here is yet another entry in our endlessly growing collection of filtering examples, highlighting yet another hitherto unexplored aspect, combining a view filter with a parameter filter for a specific phase.

**Question:** I need to retrieve all elements in a specific view that have same phase, created but not demolished.
How can I achieve that, please?

**Answer:** To get all elements in a specified view, you can use the overload of the FilteredElementCollector constructor taking a view element id as a second argument.
Quoting the Revit API help file RevitAPI.chm, it 'constructs a new FilteredElementCollector that will search and filter the visible elements in a view'.

Since 'Phase Created' is a parameter, you can then use the ElementParameterFilter class to filter out elements having the specific phase.
Combining the two filters will provide your target element set.

Here is some sample code demonstrating this:
```python
[TransactionAttribute( TransactionMode.Manual )]
[RegenerationAttribute( RegenerationOption.Manual )]
public class RevitCommand : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    Transaction trans = new Transaction( doc, "Test" );
    trans.Start();

    // use the view filter

    FilteredElementCollector collector
      = new FilteredElementCollector(
        doc, doc.ActiveView.Id );

    // use the parameter filter.
    // get the phase id "New construction"

    ElementId idPhase = GetPhaseId(
      "New Construction", doc );

    ParameterValueProvider provider
      = new ParameterValueProvider(
        new ElementId( ( int )
          BuiltInParameter.PHASE\_CREATED ) );

    FilterNumericRuleEvaluator evaluator
      = new FilterNumericEquals();

    FilterElementIdRule rule
      = new FilterElementIdRule(
        provider, evaluator, idPhase );

    ElementParameterFilter parafilter
      = new ElementParameterFilter( rule );

    collector.WherePasses( parafilter );

    TaskDialog.Show( "Element Count",
      "There are " + collector.Count().ToString()
      + " elements in the current view created"
      + " with phase New Construction" );

    trans.Commit();

    return Result.Succeeded;
  }

  public ElementId GetPhaseId(
    string phaseName,
    Document doc )
  {
    ElementId id = null;

    FilteredElementCollector collector
      = new FilteredElementCollector( doc );

    collector.OfClass( typeof( Phase ) );

    var phases = from Phase phase in collector
                 where phase.Name.Equals( phaseName )
                 select phase;

    id = phases.First().Id;

    return id;
  }
}
```