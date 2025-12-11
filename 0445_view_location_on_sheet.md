---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.9
content_type: code_example
optimization_date: '2025-12-11T11:44:13.956949'
original_url: https://thebuildingcoder.typepad.com/blog/0445_view_location_on_sheet.html
post_number: '0445'
reading_time_minutes: 9
series: views
slug: view_location_on_sheet
source_file: 0445_view_location_on_sheet.htm
tags:
- csharp
- elements
- filtering
- levels
- parameters
- python
- revit-api
- rooms
- selection
- sheets
- transactions
- views
title: View Location on Sheet
word_count: 1828
---

### View Location on Sheet

I returned from a beautiful little mountain hike to the lake, pass and mountain of
[Piz Lunghin](http://en.wikipedia.org/wiki/Piz_Lunghin).

![Jeremy on Piz Lunghin](file:////j/photo/jeremy/2010/2010-09-22_lunghin/p9204277_jeremy_on_lunghin.jpg)

One very special aspect of this place is that it is Europe's one and only triple watershed, with rivers flowing all the way to the North Sea, Black Sea and Mediterranean.
Another special aspect of my hike was spending the night outside on the mountain, with a wonderful clear sky and the almost full moon in sub-zero temperatures (Celsius).

Returning once again to the Revit API, the SDK sample AllViews collects all views in the model, asks the user to interactively select some of them, and places those on a sheet.

Even so, the exact definition and calculation of the view coordinates and location of the view on the sheet remains somewhat obscure.
Here are some helful results in this area.
They do not answer every possible question in full completeness, so we still have some work to do exploring this area, but hopefully they will be useful already in this incomplete form.

We recently had one
[question](http://thebuildingcoder.typepad.com/blog/2009/01/viewports-and-sheets.html?cid=6a00e553e1689788330134863f0fed970c#comment-6a00e553e1689788330134863f0fed970c) from
Thomas Fink of
[SOFiSTiK](http://www.sofistik.de) on
this, and here is another recent question on that topic with some interesting research performed by my colleague Joe Ye.
I am reproducing the rather long discussion in full, to show that the issue is not trivial and the solution is not yet completely perfect:

**Question:** Given a certain view placed on a sheet, I am trying to retrieve the location of the view on the sheet using:
```csharp
  Location loc = Element.Location;
```
This is not giving me what I need, which is the UV or XYZ location of the view placed on the sheet.

**Answer:** You can use the Element BoundingBox property to retrieve the Max and Min points of a viewport in the sheet.
From these, you can calculate the middle point of the Max and Min point.
The middle point is the viewport's centre point in sheet.

Here is some code showing to determine the centre point of a view in a sheet view.
To quickly test the solution, it currently uses a hardcoded element id 157165 for the viewport in the sheet view; please change that to suit your model and needs.
In addition, please pass in the sheet object as an argument in the call to Element.get\_BoundingBox.
In this sample, we just set the sheet view to be the active view:
```python
[TransactionAttribute( TransactionMode.Manual )]
[RegenerationAttribute( RegenerationOption.Manual )]
public class GetViewPosition : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string messages,
    ElementSet elements )
  {
    UIApplication app = commandData.Application;
    Document doc = app.ActiveUIDocument.Document;

    ElementId id = new ElementId( 157165 );
    Element elem = doc.get\_Element( id );

    BoundingBoxXYZ bounding
      = elem.get\_BoundingBox( doc.ActiveView );

    string sMsg = "The Max and Min points are: \n";

    sMsg += bounding.Max.X.ToString()
      + ", " + bounding.Max.Y.ToString()
      + ", " + bounding.Max.Z.ToString() + "\n";

    sMsg += bounding.Min.X.ToString()
      + ", " + bounding.Min.Y.ToString()
      + ", " + bounding.Min.Z.ToString() + "\n";

    XYZ xyzPosition
      = ( bounding.Max + bounding.Min ) / 2.0;

    sMsg += "view position in current sheet is:\n"
      + xyzPosition.X.ToString()
      + "," + xyzPosition.Y.ToString()
      + "," + xyzPosition.Z.ToString();

    MessageBox.Show( sMsg );

    return Result.Succeeded;
  }
}
```

**Response:** I need to programmatically copy a legend view from one sheet to another.
For this, I need to get the location of the view from the original sheet and reuse this location when adding new views to the other sheets.

For this purpose, I am using the following code:
```csharp
  foreach( ViewSheet chosenSheet in SelectedSheets )
  {
    BoundingBoxXYZ xyzLocation
      = selectedView.get\_BoundingBox(
        m\_Doc.ActiveView );

    XYZ xyzPosition = ( xyzLocation.Max
      + xyzLocation.Min ) / 2.0;

    int scale = correspondingView.Scale;

    chosenSheet.AddView( correspondingView,
      new UV( xyzPosition.X, xyzPosition.Y ) );
  }
```

Here I am using the following variables:

- chosenSheet is the sheet where the legend will be copied.- selectedView is the selected Viewport to be copied.- correspondingView is the view to be copied.

Unfortunately, the views are not copied to the same location.

**Answer:** I created a simple project and added several view sheets.
I picked the Level 1 viewport and tried to run this code, but this caused an error message saying 'Added view is already on another sheet and cannot be displayed on two sheets'.

**Response:** This is because you are trying to copy a "plan" view that is already placed on a sheet and place it on another sheet.
Try running this code on a legend view placed on a sheet.

**Answer:** The point used to insert the view on the view sheet is mapping the (0,0,0) point of the source view.
When you retrieve the centre point coordinates of the viewport, they do not match the (0,0,0) point of the source view.
So you need to calculate the source view's original point coordinates in the view sheet from the viewpoint's centre point coordinates.

**Response:** I understand that I need to get the centre of the view in the ViewSheet in order to be able to calculate the shift.
How can I obtain the coordinates of the centre of the view relative to the view sheet?

**Answer:** The View class has a very important property Outline.
The Outline property will return the Max and Min point of the closest bounding box, which includes all elements in this view.
The value in this Max and Min the result of the legend view's real Max and Min coordinates subdivided by the view scale.
For more information about Outline property, please refer to the Revit SDK developer guide 'Revit 2011 API Developer Guide.pdf'.

Approximately, the outline Max point is mapping the same Max point of the viewport in view sheet.
So we can calculate the source view's origin coordinates in the host view sheet using the following code:
```csharp
  BoundingBoxXYZ xyzLocation
    = SelectedView.get\_BoundingBox(
      m\_Doc.ActiveView );

  // get the outline max and min

  XYZ ptMaxOutline = new XYZ(
    CorrespondingView.Outline.Max.U,
    CorrespondingView.Outline.Max.V,
    0 );

  // get the view's origin point's
  // coordinates in current view sheet.

  UV ptSourceViewOriginInSheet = new UV(
    xyzLocation.Max.X - ptMaxOutline.X,
    xyzLocation.Max.Y - ptMaxOutline.Y );
```

Here is the complete code of the Execute method:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  try
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Application app = uiapp.Application;
    Document doc = uidoc.Document;

    SelElementSet selection = uidoc.Selection.Elements;
    if( selection.Size == 0 )
    {
      MessageBox.Show( "Please Select a view to be copied.",
        "View Coordinates", MessageBoxButtons.OK,
      MessageBoxIcon.Error );
      return Result.Failed;
    }
    else
    {
      // Get all Sheets
      List AllSheets = GetAllSheets( doc );

      // Get all Views in the model.
      // Find all views in the document by using category filter

      ElementCategoryFilter filter
        = new ElementCategoryFilter(
          BuiltInCategory.OST\_Views );

      FilteredElementCollector collector
        = new FilteredElementCollector( doc );

      IList AllViews = collector.WherePasses( filter )
        .ToElements();

      foreach( Element SelectedView in selection )
      {
        if( SelectedView.Category.Name == "Viewports" )
        {
          String ViewName = SelectedView.get\_Parameter(
            "View Name" ).AsString();

          View CorrespondingView = null;

          for( int i = 0; i < AllViews.Count; i++ )
          {
            if( AllViews[i].Name == ViewName )
            {
              CorrespondingView = AllViews[i] as View;
              break;
            }
          }

          BoundingBoxXYZ xyzLocation
            = SelectedView.get\_BoundingBox(
              doc.ActiveView );

          // get the outline max and min

          XYZ ptMaxOutline = new XYZ(
            CorrespondingView.Outline.Max.U,
            CorrespondingView.Outline.Max.V, 0 );

          // get the view's origin point's coordinates
          // in current view sheet.

          UV ptSourceViewOriginInSheet = new UV(
            xyzLocation.Max.X - ptMaxOutline.X,
            xyzLocation.Max.Y - ptMaxOutline.Y );

          // for each chosen to be exported
          // get the views on it.

          foreach( ViewSheet ChosenSheet in AllSheets )
          {
            ChosenSheet.AddView( CorrespondingView,
              ptSourceViewOriginInSheet );
          }
        }
      }
      return Result.Succeeded;
    }
  }
  catch( Exception ex )
  {
    MessageBox.Show( ex.Message );
    return Result.Failed;
  }
}
```

**Response:** Using this code, the location of the view on the sheet is still inaccurate.
If I use this code to duplicate a view legend placed on a sheet in the same location, the legend view still appears to be shifted while duplicated on the same sheet, as shown in the following figures.
Here is a view of the entire sheet:

![View locations offset on sheet](img/view_loc_on_sheet1.png)

Here is a more detailed picture of the offset between the original and copied views:

![View locations offset](img/view_loc_on_sheet2.png)

**Answer:** The reason for this might be that the Revit API returns the Viewport BoundingBox property value with an offset to its original outline property.
The BoundingBox property is not completely accurate, so the result to calculate the legend view's origin in the sheet view cannot be so either.
As far as I can tell, the offset of the Max point of the viewport's BoundingBox is enlarged 0.01 in both X and Y direction.
We can take this scaling into account.

Here is some updated code with an adjustment added.
You might need to further adjust the offset value according to different situations.
The offset value of 0.01 is not obtained from source code, but from my empirical testing results:
```csharp
foreach( Element SelectedView in selection )
{
  if( SelectedView.Category.Name == "Viewports" )
  {
    string ViewName = SelectedView.get\_Parameter(
      "View Name" ).AsString();

    View CorrespondingView = null;

    for( int i = 0; i < AllViews.Count; i++ )
    {
      if( AllViews[i].Name == ViewName )
      {
        CorrespondingView = AllViews[i] as View;
        break;
      }
    }

    BoundingBoxXYZ xyzLocation
      = SelectedView.get\_BoundingBox(
        doc.ActiveView );

    // get the outline max and min,
    // offset in each direction of max point of
    // bounding box than the outline box.

    double dMaxOffset = 0.01;
    XYZ ptMaxOutline = new XYZ(
      CorrespondingView.Outline.Max.U,
      CorrespondingView.Outline.Max.V, 0 );

    UV ptSourceViewOriginInSheet = new UV(
      xyzLocation.Max.X - dMaxOffset - ptMaxOutline.X,
      xyzLocation.Max.Y - dMaxOffset - ptMaxOutline.Y );

    // for each chosen to be exported get the views on it

    foreach( ViewSheet ChosenSheet in AllSheets )
    {
      ChosenSheet.AddView( CorrespondingView,
        ptSourceViewOriginInSheet );
    }
  }
}
```

Thank you very much, Joe, for this thorough exploration!

Here is another quick question and answer that might be of use in this context:

#### Centre Point of View

**Question:** I need a way to get the XYZ of the centre of a view and the XYZ of the centre of a sheet. Can I achieve this using one of the Outline, CropBox or BoundingBox properties?

**Answer:** It depends on what exactly you mean by 'centre of view'.
I can imagine that it might be any one of the following:

- The visible centre of a view:
  If we pan the view, the centre coordinates change.
  There is currently no API exposed to get the visible centre of a view, and we do have an open wish list item to obtain this information.- The centre point of the crop box:
    We can retrieve the BoundingBoxXYZ of the view through the CropBox property.
    Then the centre can be determined by Max and Min of the crop box.- The coordinate centre:
      the coordinate centre point is returned by the View.Origin property.

For an example of defining a completely different set of view coordinates, we also once looked at
[cropping a 3D view to a specific element](http://thebuildingcoder.typepad.com/blog/2009/12/crop-3d-view-to-room.html).