---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: code_example
optimization_date: '2025-12-11T11:44:14.336501'
original_url: https://thebuildingcoder.typepad.com/blog/0653_activate_3d_view.html
post_number: '0653'
reading_time_minutes: 2
series: views
slug: activate_3d_view
source_file: 0653_activate_3d_view.htm
tags:
- elements
- filtering
- levels
- parameters
- python
- revit-api
- transactions
- views
title: Activate a 3D View
word_count: 443
---

### Activate a 3D View

Here is a simple question on making use of the ActiveView setter, which is new in Revit 2012:

**Question:** I heard that Revit 2012 exposes an API to change the active view.

I would like to change the active view to 3D with the Detail Level set to Fine and Visual Style set to Realistic.

Could you please show me the correct way to change the active view to the one I want?

**Answer:** I already discussed
[setting the visual style for a view](http://thebuildingcoder.typepad.com/blog/2011/06/set-the-visual-style-of-a-view.html),
so we can make use of that here.

To answer the rest of your query, I created a little sample application for you which says (and does) it all.

It is short and sweet, so I can present it here in its entire glory;
first, here is a helper method Get3dView to retrieve a suitable 3D view using a filtered element collector:
```python
  /// <summary>
  /// Retrieve a suitable 3D view from document.
  /// </summary>
  View3D Get3dView( Document doc )
  {
    FilteredElementCollector collector
      = new FilteredElementCollector( doc )
        .OfClass( typeof( View3D ) );

    foreach( View3D v in collector )
    {
      Debug.Assert( null != v,
        "never expected a null view to be returned"
        + " from filtered element collector" );

      // Skip view template here because view
      // templates are invisible in project
      // browser

      if( !v.IsTemplate )
      {
        return v;
      }
    }
    return null;
  }
```

The external command Execute mainline makes use of this method to determine and activate the 3D view to use and set up the Detail Level and Visual Style parameters:
```python
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
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

    // Find a suitable 3D view:

    View3D view = Get3dView( doc );

    if( null == view )
    {
      message = "Sorry, no suitable 3D view found";

      return Result.Failed;
    }
    else
    {
      // Must set view before starting the transaction,
      // otherwise an exception is thrown saying
      // "Cannot change the active view of a modifiable
      // document (with a transaction currently open)."

      uidoc.ActiveView = view;

      // Must start a transaction in order to set the
      // parameters on the view:

      Transaction t = new Transaction( doc );
      t.Start( "Change to 3D view" );

      view.get\_Parameter( BuiltInParameter
        .VIEW\_DETAIL\_LEVEL ).Set( 3 );

      view.get\_Parameter( BuiltInParameter
        .MODEL\_GRAPHICS\_STYLE ).Set( 6 );

      t.Commit();

      return Result.Succeeded;
    }
  }
}
```

The one and only slightly tricky part is that writing to the ActiveView property to switch the view requires no transaction to be active, whereas setting the parameters does.

For completeness sake, here is
[ChangeTo3dView.zip](zip/ChangeTo3dView.zip) containing
the entire project, source code and add-in manifest file for this command.