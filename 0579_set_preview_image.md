---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.206984'
original_url: https://thebuildingcoder.typepad.com/blog/0579_set_preview_image.html
post_number: 0579
reading_time_minutes: 5
series: views
slug: set_preview_image
source_file: 0579_set_preview_image.htm
tags:
- csharp
- elements
- family
- filtering
- revit-api
- sheets
- transactions
- views
title: Set Preview Image
word_count: 948
---

### Set Preview Image

While I am
[away](http://thebuildingcoder.typepad.com/blog/2011/05/improved-mep-element-shape-and-mount-ararat.html#2),
here is a useful article to keep you occupied by my colleague Saikat Bhattacharya:

### Setting preview image/view for a Revit document using Revit 2012 API

**Question:** How can I set the preview image/view for a Revit document using the Revit 2012 API?
In particular, when I load an older version of Revit document and do a SaveAs using the API, how can I set its preview image?

**Answer:** There are two ways to set a preview image on a Revit document using the API.

The first approach uses DocumentPreviewSettings to assign a permanent view to the document to always be used for preview settings. This is done by serializing this preview id into the document. Using this approach, the preview view will be permanently set for the document.

In the user interface, when we save a file, typically a permanent view for the preview generation is not assigned. It is only assigned if the users click on the Options button in the SaveAs dialog and select a view other than 'Active view/sheet' in the Preview Source of the dialog. It is this permanent view that can be extracted and set using the DocumentPreviewSettings API. If there is no permanent preview view assigned here, Revit will use the default active view/sheet to generate the preview.

The other approach set a temporary preview view id which will be used during the single save operation and is not serialized to the document. This can be done using the Document.Save(SaveOptions) or Document.SaveAs(SaveAsOptions). Both these option classes have a setting for the preview view id.

In a typical use case of OpenDocumentFile(), performing certain changes to the document and then saving the file either using a Save or SaveAs results in Revit having no way to know how to save a preview and so previews are lost. This is where Save(SaveOptions) or SaveAs(SaveAsOptions) helps. However, setting the temporary preview view settings using the save options can become challenging since API users would still not know which view to be set as preview – the Document.ActiveView does not work for an un-displayed document. Here, there are two ways of approaching this:

**1.** Using a new API called StartingViewSettings, which helps access the initial view settings for a document which returns the view that will be initially opened when a model is opened. This view is usually set by the user and can be accessed from the user interface by accessing the 'Manage' tab in which we need to click on the 'Starting View' button. If API users get a valid view from the StartingViewSettings and end up using this for setting the preview view, it is recommended that this view be set as preview using the temporary setting (which is to use the Save(SaveOptions) or the SaveAs(SaveAsOptions) method and not the DocumentPreviewSettings. StartingViewSettings works only for Revit projects and will throw an exception if used with a Revit family file (RFA). If the starting view has not been set by the user, we will see '<Last Viewed>' as the current settings. On the API side, this means that the StartingViewSettings.ViewId will return a InvalidElementId in which case we need to follow the second approach.

**2.** The other approach is to make an algorithmic choice e.g. finding the 3D view (ensuring that this is not a template) and using this view as temporary preview view setting. If no 3D views are found, then the API user can decide on using plan views instead, etc. If you end up using this algorithmic approach for setting the preview view, it is recommended that the preview be set using the temporary setting (which is to use the Save(SaveOptions) or the SaveAs(SaveAsOptions) method and not the DocumentPreviewSettings.

The following lines of code show a typical use of the various APIs which help in setting the preview.
It covers all the various approaches mentioned above:
```csharp
  // Open Revit file
  Document doc = cmdData.Application.Application
    .OpenDocumentFile( file.FullName );

  // Create transaction and start
  Transaction trans = new Transaction( doc, "T1" );
  trans.Start();

  doc.Regenerate();

  // Create thumbnails for preview
  DocumentPreviewSettings previewSettings =
    doc.GetDocumentPreviewSettings();

  SaveAsOptions saveOpts = new SaveAsOptions();

  // Check for permanent preview view
  if( doc
    .GetDocumentPreviewSettings()
    .PreviewViewId
    .Equals( ElementId.InvalidElementId ) )
  {
    // Access the intial view
    StartingViewSettings startingViewSettings
      = StartingViewSettings.GetStartingViewSettings(
        doc );

    if( !startingViewSettings.ViewId.Equals(
      ElementId.InvalidElementId ) )
    {
      // If valid, then set the viewId
      //previewSettings.PreviewViewId
      //  = startingViewSettings.ViewId;

      saveOpts.PreviewViewId
        = startingViewSettings.ViewId;
    }
    else
    {
      // Algorithmic approach - look for 3D views

      FilteredElementCollector collector
        = new FilteredElementCollector( doc )
          .OfClass( typeof( View ) );

      IEnumerable views
        = from Autodesk.Revit.DB.View f
          in collector
          where ( f.ViewType == ViewType.ThreeD
            && !f.IsTemplate )
          select f;

      bool bNotFound = true;

      foreach( View vw in views )
      {
        if( !vw.IsTemplate )
        {
          //If we were to set the preview view permanently,
          // we would use the following two commented lines
          //previewSettings.PreviewViewId = vw.Id;
          //previewSettings.ForceViewUpdate( true );

          // Set the temporary view
          saveOpts.PreviewViewId = vw.Id;
          bNotFound = false;
          break;
        }
      }

      // If no 3D view is obtained, look for other valid views
      if( bNotFound )
      {
        IEnumerable viewsNon3D
          = from View fNon3D
            in collector
            where ( fNon3D.ViewType == ViewType.FloorPlan
              || fNon3D.ViewType == ViewType.EngineeringPlan
              || fNon3D.ViewType == ViewType.Elevation
              || fNon3D.ViewType == ViewType.Section
              && !fNon3D.IsTemplate )
            select fNon3D;

        foreach( View vw in viewsNon3D )
        {
          if( !vw.IsTemplate )
          {
            // If we were to set the preview view permanently,
            // we would use the following two commented lines
            //previewSettings.PreviewViewId = vw.Id;
            //previewSettings.ForceViewUpdate( true );

            saveOpts.PreviewViewId = vw.Id;
            break;
          }
        }
      }
    }
  }

  // Commit transaction
  trans.Commit();

  // Save Revit file to target destination
  doc.SaveAs( destPath + "\\" + file.Name, saveOpts );

  // Close document
  doc.Close();
```

Many thanks to Saikat for this useful explanation!