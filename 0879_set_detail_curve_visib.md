---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.1
content_type: code_example
optimization_date: '2025-12-11T11:44:14.787753'
original_url: https://thebuildingcoder.typepad.com/blog/0879_set_detail_curve_visib.html
post_number: 0879
reading_time_minutes: 2
series: geometry
slug: set_detail_curve_visib
source_file: 0879_set_detail_curve_visib.htm
tags:
- csharp
- elements
- family
- filtering
- levels
- parameters
- python
- references
- revit-api
- selection
- transactions
- views
- geometry
title: Set Detail Curve Visibility
word_count: 425
---

### Set Detail Curve Visibility

Mostly, the Revit API limits an add-in to pretty high-level operations, encapsulating and protecting the parametric BIM from reckless modifications.
For the nonce, here is a little bit of nitty-gritty bit manipulation anyway, in a utility method provided by Scott Conover:

**Question:** Why is there no DetailCurve.SetVisibility method exposed, similar to ModelCurves.SetVisibility?

The UI has this functionality, and I would find very good use for it in reinforcement detailing.

I want to programmatically create a detail family representing a certain rebar shape.
The family instance should reflect the visibility states, single line in coarse, double line in fine.

**Answer:** The ModelCurve.SetVisibility method taking a FamilyElementVisibility argument is available for visibility of curve elements in families when the families are placed.
Similarly we have a SymbolicCurve.SetVisibility method.
Detail curves don't have the exact same functionality in the UI.
Generally, detail curves are visible only in a single view in which they are placed.
The access you describe is partially enabled for detail families, however.

Here is a workaround to access this functionality: the visibility settings for the curve are stored in the integer GEOM\_VISIBILITY\_PARAM built-in parameter as bit flags.
You can therefore request the desired display by setting the appropriate bits in that parameter value.

The SetFamilyVisibility method presented below turns off the modes you do not want for visibility.
It can be called either from an external command or a macro.

It makes use of a simple selection filter to restrict the user selection to detail curves:
```python
class DetailCurveSelectionFilter : ISelectionFilter
{
  public bool AllowElement( Element e )
  {
    CurveElementFilter filter
      = new CurveElementFilter(
        CurveElementType.DetailCurve );

    return filter.PassesFilter( e );
  }

  public bool AllowReference( Reference r, XYZ p )
  {
    return false;
  }
}
```

Here is the method implementation itself:
```csharp
public void SetFamilyVisibility( UIDocument uidoc )
{
  Document doc = uidoc.Document;

  Reference r = uidoc.Selection.PickObject(
    ObjectType.Element,
    new DetailCurveSelectionFilter(),
    "Select detail curve" );

  Element elem = doc.GetElement( r );

  Parameter visParam = elem.get\_Parameter(
    BuiltInParameter.GEOM\_VISIBILITY\_PARAM );

  int vis = visParam.AsInteger();

  using( Transaction t = new Transaction( doc ) )
  {
    t.Start( "Set curve visibility" );

    // Turn off the bit corresponding
    // to the unwanted modes

    vis = vis & ~( 1 << 13 ); // Coarse
    //vis = vis & ~(1 << 14); // Medium
    //vis = vis & ~(1 << 15); // Fine

    visParam.Set( vis );

    t.Commit();
  }
}
```

Remember that all three modes cannot be turned off simultaneously – a posted error will result.

Here is
[SetDetailCurveVisibility.zip](zip/SetDetailCurveVisibility.zip) containing
the complete source code, Visual Studio solution and add-in manifest for an external command implementation of this.

Many thanks to Scott for this tricky hint!