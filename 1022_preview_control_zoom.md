---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:15.141394'
original_url: https://thebuildingcoder.typepad.com/blog/1022_preview_control_zoom.html
post_number: '1022'
reading_time_minutes: 8
series: views
slug: preview_control_zoom
source_file: 1022_preview_control_zoom.htm
tags:
- csharp
- elements
- levels
- references
- revit-api
- views
- windows
title: AppStore Advice and Zooming in a Preview Control
word_count: 1554
---

### AppStore Advice and Zooming in a Preview Control

Let's look at another quick
[PreviewControl question](#4).

First, however, please note:

#### Portathon Day 2

Today is the second day of the Portathon!

- [Earn $100 Submitting an Autodesk Exchange App](http://thebuildingcoder.typepad.com/blog/2013/07/earn-100-submitting-an-autodesk-exchange-app.html)
- [Exchange App Videos](http://thebuildingcoder.typepad.com/blog/2013/08/exchange-app-videos-and-devtv-youtube-channel.html)
- [Portathon reminder and video](http://thebuildingcoder.typepad.com/blog/2013/08/adn-training-material-on-github-and-portathon-reminder.html#2)
- [Cloud and AppStore usage growth](http://thebuildingcoder.typepad.com/blog/2013/09/cloud-and-appstore-usage-grows-portathon-reminder.html)

Everything went well yesterday, and today we continue.

Last chance to earn your hundred bucks!

#### Advice on Submitting an App to the Exchange Store

While we are on the topic of the exchange store and Portathon event, let me mention some points that arose in one case we handled yesterday.

I recently mentioned Paul Kinnane's enthusiastic
[feedback on the custom exporter](http://thebuildingcoder.typepad.com/blog/2013/09/access-to-individual-elements-in-linked-projects.html#5).

Paul went on to submit his Octane renderer add-in as an app to the Autodesk Exchange Store and ran into some issues which triggered a couple of recommendations that seem worthwhile sharing:

**Question:** I am in the process of migrating the plugin to the Store compatible .bundle format instead of loading from `C:\ProgramData\Autodesk\Revit\Addins\2014`.
However, I cannot get the plugin to load into Revit when the files are in `C:\ProgramData\Autodesk\ApplicationPlugins\OctaneRender for Revit.bundle`.

In summary – when all the required .addin, .dat and .dll files are in `C:\ProgramData\Autodesk\Revit\Addins\2014`, the plugin is correctly added to the Add-Ins toolbar. But copying those files into the above .bundle folder results in the plugin not appearing on the toolbar. No error is raised. No amount of adding or editing xml files, putting the DLLs in Contents, etc. seems to help. I watched all the Autodesk videos on this and read all the docs. Can you advise how I can best resolve this issue pls?

**Answer:** Do you plan to submit your plugin as an exchange store app this week?

If so, let's make sure we get it fixed during the Portathon at the latest.

Have you downloaded and installed any other apps? You could use them to compare with own your setup.

Does your add-in no longer load at all, or is it just a problem that it does not load dynamically during runtime, without restarting Revit?

I heard of an issue regarding the new AllowLoadIntoExistingSession tag in the add-in manifest file, so it would be good to ensure that your problem is different.

Another thing to check is your XML file encoding:
In my experience, you can use
[any encoding you want in the Revit add-in manifest](http://thebuildingcoder.typepad.com/blog/2010/12/snow-and-woe-with-manifest-files.html), but it must correspond to the coding specified in the corresponding XML file tag, i.e.,
[the tag must tell the truth](http://thebuildingcoder.typepad.com/blog/2010/04/addin-manifest-and-guidize.html#3).

**Response:** Thanks for all the suggestions Jeremy.

The plugin does not load even after a Revit restart.

It may be my Revit knowledge, but I found getting the plugin to run under the other Autodesk CAD packages to be relatively straightforward – and they also provide a reasonable level of diagnosis, whereas with Revit I was unable to determine if the plugin had in fact loaded or not. This is compounded by not being able to run Revit directly from the Visual Studio debugger (it crashes), so there is a need to run Revit from the desktop then attach in Visual Studio, by which time you may have missed some debug opportunities when the plugin loads.

AllowLoadIntoExistingSession is not present in the add-in file.

My suspicion is that the plugin does not load from the `C:\ProgramData\Autodesk\ApplicationPlugins` folder because Revit is not loading the required DLLs when it's in that folder, whereas they are loaded from `C:\ProgramData\Autodesk\Revit\Addins\2014`. But diagnosing this is very difficult. The other possibility is as you say, that it could be an XML issue.

To add to my confusion, I downloaded your latest OBJ exporter, compiled it, copied (dll & addin) into `C:\ProgramData\Autodesk\Revit\Addins\2014` and the ObjExport.dll did not load. I had to put ObjExport.dll into the Revit 2014 executable folder to get it to load.

There needs to be a log file to indicate what is and is not being loaded, and if it's not being loaded, why.

Thanks again for your suggestions and comments.
Very much appreciated.

**Answer:** Sorry to hear about your struggles, and thank you for your appreciation.

I have no issues launching Revit from the Visual Studio 2010 debugger.
When I do that, I can see my add-in being loaded by Revit in the debug output window.

Just copying the .NET assembly DLL and the add-in manifest to the ApplicationPlugins is not sufficient.
The add-in manifest is only read when you place it in the Addins folder.
The ApplicationPlugins location requires a PackageContents.xml file instead.
So you must have missed one or two small details in the docs after all :-)

Anyway, as you presumably know, we are running the Autodesk Exchange Store
[Portathon event today](http://thebuildingcoder.typepad.com/blog/2013/09/multireferenceannotation-example.html).

I therefore passed on your query to both the Revit API development team and my colleagues dealing with the exchange store, and they reply as follows:

1. Do you have a PackageContents.xml file in the bundle folder?
   Revit is configured to look for Revit apps inside the PackageContents.xml files in this location.
   It is not configured to directly load .addin files from there.
   If the plugin is working from `C:\ProgramData\Autodesk\Revit\Addins\2014`, which seems to be the case, and you are ready to submit it to the appstore, we would generate a PackageContents.xml file for you and you can test it from there.
2. We created an installer using the files submitted by the publisher. It seems to be working fine. The add-in is getting loaded for Revit 2014. The bundle will be placed in `%ProgramData%\Autodesk\ApplicationPlugin`.
   We will be testing the app next week and contact the publisher regarding this directly.

I hope this matches you intentions!

**Response:**
Thank you Jeremy.
Yes that very much matches our intentions.

Back to the main topic:

#### Zooming in a Preview Control

**Question:** I have a question about the preview control.

I am displaying a solid paint by AVF in a 3D view in the preview control.
Painting the solid works fine.

However, I am having an issue trying to zoom and centre once the preview control is loaded.

My code looks something like this:

```csharp
  previewControl.Loaded += new RoutedEventHandler(
    PreviewControl\_Loaded);

  // . . .

  private void PreviewControl\_Loaded(
    object sender,
    EventArgs args)
  {
    var pvControl = sender as PreviewControl;
    var viewBoundingBox = \_solid.GetBoundingBox();
    pvControl.UIView.ZoomAndCenterRectangle(
      viewBoundingBox.Min, viewBoundingBox.Max);
  }
```

It doesn't work at all.

If I add a button and register a separate click event to call the UIView ZoomAndCenterRectangle method with the min and max points of the solid bounding box, it works as I want.

It seems that the UIView method will only work after the view is displayed in the preview control.

Is there any way to zoom the preview control before displaying the view?

**Answer:** Cool.

I am somewhat surprised and impressed to hear that you are able to zoom in the view after the control has been displayed.

I was under the impression that you had no programmatic access to control the preview control at all once it has been instantiated.

I thought the only programmatic control was the instantiation itself, when you pass in the document and view id.

If you wish the preview control to be automatically zoomed to a specific region before it is displayed, I would suggest zooming the view that you pass in to the control correctly **before** instantiating it.

For the sake of completeness, here are pointers to the previously available preview control samples and resolved issues:

- [Preview control](http://thebuildingcoder.typepad.com/blog/2012/03/revit-2013-and-its-api.html#2)
- [DevCamp session on the Revit 2013 UI API enhancements](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#2)
- [PreviewControlSimple](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-two.html#23)
- [UIAPI Revit SDK sample](http://thebuildingcoder.typepad.com/blog/2012/03/new-revit-2013-sdk-samples.html#4)
- [Preview control with linked document](http://thebuildingcoder.typepad.com/blog/2012/08/vacation-time-and-various-notes.html#4)
- [Using a preview control in a macro](http://thebuildingcoder.typepad.com/blog/2013/08/issue-using-a-preview-control-in-a-macro.html)

#### Vacation Time

The Portathon event is my last activity here before heading off on vacation for ten days.

I will be taking the computer with me, but I do not expect to publish anything or answer any comments before I return.

Take care and enjoy yourselves while I am away!