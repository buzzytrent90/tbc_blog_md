---
post_number: "0837"
title: "Detach Workset and Task Dialogue Command Link Order"
slug: "detach_workset"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'parameters', 'revit-api']
source_file: "0837_detach_workset.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0837_detach_workset.html"
---

﻿

### Detach Workset and Task Dialogue Command Link Order

Here is a neat and simple solution using the DialogBoxShowing event to detach and discard worksets by Erik Eriksson of
[White Arkitketer AB](http://www.white.se),
who already provided a useful workaround to
[synchronize with central](http://thebuildingcoder.typepad.com/blog/2012/01/synchronize-with-central.html) using
SendKeys in combination with the Idling and DocumentSynchronizedWithCentral events.

Here is the issue:

**Question:** We have a need to detach project files without worksets.

For instance, it would be great to just have an option to detach and discard worksets.

Why do we need this?

This would be useful for our in-house tools to diagnose project files.
For this to work, we need to have the project files without worksets, because we delete different types of elements and check the file size before and afterwards.
To determine a correct file size, we need to compact the files after every operation.
This cannot be done programmatically, but, for non-workshared files, it is performed automatically each time the project file is saved.

Currently, we execute the detach and discard operation manually.
However, with big projects containing many files, it would be nice to do less of these things manually.

**Answer:** The API currently does not provide this functionality directly.

However, the Revit 2013 API added some new document handling and worksharing changes which may help.
Here is the description from the What's New section in the Revit API help file RevitAPI.chm:

##### Worksharing properties

The information required to identify a workshared file on the central or local locations has changed due to changes to RevitServer.
As a result, the members

- Application.GetRevitServerNetworkHosts- Application.CurrentRevitServerAccelerator- Document.WorksharingCentralFilename

replace the properties

- Application.CentralServerName- Application.LocalServerName- Document.GetWorksharingCentralModelPath

The new members:

- Application.GetWorksharingCentralGUID(ServerPath serverModelPath)- Document.WorksharingCentralGUID

provide read access to the worksharing central GUID of the given server-based model.
This is applicable only to workshared models saved in Revit 2013 or later.

##### New overloads for Application.OpenDocumentFile and UIApplication.OpenAndActivateDocument

The new overloads support parameters OpenOptions and OpenOptionsForUI, respectively, to specify how a Revit document should be opened.
Both options classes currently offer the ability to detach the opened document from central if applicable.

- The OpenOptions.DetachFromCentralOption property can be set to the default DoNotDetach or to DetachAndPreserve.- The OpenOptionsForUI.DetachFromCentralOption property can be set to the default DoNotDetach, DetachAndPreserveWorksets or DetachAndPrompt.

In other words, the OpenOptions argument that you pass in to the OpenDocumentFile and OpenAndActivateDocument methods can now use the DetachFromCentral option to specify whether or not a workset-enabled document is detached from its central document.

**Response:** The new API functionality still does not provide the option of discarding the worksets.

I want the file to be a normal project file again.

OpenOptions holds the same options as OpenOptionsForUI (besides DetachAndPrompt).
However, maybe I can catch that dialogue programmatically using the DialogBoxShowing event and override the result...

Success!

This is the code I used to fix it:
```csharp
  public void OnDialogBoxShowing(
    object sender,
    DialogBoxShowingEventArgs e )
  {
    TaskDialogShowingEventArgs e2
      = e as TaskDialogShowingEventArgs;

    if( null != e2 && e2.DialogId.Equals(
      "TaskDialog\_Detach\_Model\_From\_Central" ) )
    {
      e.OverrideResult(
        (int) TaskDialogResult.CommandLink2 );
    }
  }
```

The full implementation of this method actually includes a few more statements taking care of other dialogue boxes, which I omitted here.

I checked the IsWorkshared property after opening the file, and it was set to false.
This indicates that it is no longer workshared, which means that the operation to discard the worksets succeeded.
When you discard worksets, the workshared central file goes back to being a normal non-workshared project file, just like what you obtain when starting from a normal project template.

One thing I do wonder about is whether CommandLink2 is always below CommandLink1 and above CommandLink3?

**Answer:** It would appear so, since the description of the TaskDialog.AddCommandLink method clearly states that "CommandLinks will always be shown in the dialog in the order of their ids".

Also, as always, keep in mind that there is no guarantee that this will continue working in future releases.
The order of these options can change at any time, and the required functionality might even be added to the official API one of these days.

Congratulations on solving this, Erik, and thank you for sharing it and providing this helpful explanation!

#### Setting IndependentTag Label to Pick Up a Parameter Value

I'll wrap up with a quick pointer to another AEC DevBlog article by Saikat Bhattacharya on
[defining the label displayed by an IndependentTag](http://adndevblog.typepad.com/aec/2012/10/annotating-with-family-instance-parameter-data-with-independenttag-in-revit.html).
Unfortunately this does not seem to be possible programmatically.
It can however easily be achieved using an appropriate template or tag family configuration.