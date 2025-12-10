---
post_number: "0494"
title: "Find Intersecting Elements"
slug: "find_intersecting"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'python', 'references', 'revit-api', 'schedules', 'selection', 'transactions', 'views']
source_file: "0494_find_intersecting.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0494_find_intersecting.html"
---

### Find Intersecting Elements

Right now I am sitting in the Tel Aviv Ben Gurion airport, which very friendlily provides free public wifi Internet access, unlike the also very friendly staff with an unfortunately less generous wifi policy in the Sheraton hotel we stayed at, who (try to) charge USD 20 per day for the same service.

Yesterday we looked at the
[XML family usage report](http://thebuildingcoder.typepad.com/blog/2010/12/xml-family-usage-report.html) from
Kevin Vandecar's
[filtering and optimisation](http://thebuildingcoder.typepad.com/blog/2010/10/filtered-element-collectors.html) presentation
at the
[AEC DevCamp](http://thebuildingcoder.typepad.com/blog/2010/03/devcamp-devlabs-and-updated-api-training-schedule.html) conference
in Boston in June, which formed the base of my
[AU class CP234-2](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-university-2010-class-materials.html) on
the same topic.

I would like to present another of Kevin's filtering examples, which I find rather neat and demonstrates one approach to find all element in close proximity to a selected one.
This frequently requested function is easy to implement, because the filtered element collector framework offers classes providing exactly the functionality we need for this:

- The collector viewId constructor narrows down the selection to only viewable elements.- The BoundingBoxIntersectsFilter returns all elements that intersect a given bounding box.- An
      exclusion filter can be used to eliminate known elements to exclude, specifically:
      - The view itself, which often will have an intersecting bounding box.- The selected element.

Here is the short external command Execute mainline implementation putting this together:
```python
[Transaction( TransactionMode.ReadOnly )]
[Regeneration( RegenerationOption.Manual )]
public class FilterEx2 : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiApp = commandData.Application;
    UIDocument uiDoc = uiApp.ActiveUIDocument;
    Application app = uiApp.Application;
    Document doc = uiDoc.Document;

    // Select something to use as base bounding box.

    Reference r = uiDoc.Selection.PickObject(
      ObjectType.Element );

    // Find the bounding box from the selected
    // object and convert to outline.

    BoundingBoxXYZ bb = r.Element.get\_BoundingBox(
      doc.ActiveView );

    Outline outline = new Outline( bb.Min, bb.Max );

    // Create a BoundingBoxIntersectsFilter to
    // find everything intersecting the bounding
    // box of the selected element.

    BoundingBoxIntersectsFilter bbfilter
      = new BoundingBoxIntersectsFilter( outline );

    // Use a view to construct the filter so we
    // get only visible elements. For example,
    // the analytical model will be found otherwise.

    FilteredElementCollector collector
      = new FilteredElementCollector(
        doc, doc.ActiveView.Id );

    // Lets also exclude the view itself (which
    // often will have an intersecting bounding box),
    // and also the element selected.

    ICollection<ElementId> idsExclude
      = new List<ElementId>();

    idsExclude.Add( r.Element.Id );
    // No need for this, BoundingBoxIntersectsFilter
    // excludes all objects derived from View, as
    // pointed out by Jim Jia below:
    //
    //idsExclude.Add( doc.ActiveView.Id );

    // Get the set of elements that pass the
    // criteria. Note both filters are quick,
    // so order is not so important.

    collector.Excluding( idsExclude )
      .WherePasses( bbfilter );

    // Generate a report to display in the dialog.

    int nCount = 0;
    string report = string.Empty;
    foreach( Element e in collector )
    {
      string name = e.Name;

      report += "\nName = " + name
        + " Element Id: " + e.Id.ToString();

      nCount++;
    }

    Util.ShowTaskDialog(
      "Bounding Box + View + Exclusion Filter",
      "Found " + nCount.ToString()
      + " elements whose bounding box intersects",
      report );

    return Result.Succeeded;
  }
}
```

Running this in the StructuralUsage.rvt model, I select the following highlighted beam:

![Beam intersecting various elements](img/find_intersecting_0.png)

This produces the following report of intersecting elements:

![Report of intersecting elements](img/find_intersecting_1.png)