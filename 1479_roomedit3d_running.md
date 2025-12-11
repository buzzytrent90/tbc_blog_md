---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.3
content_type: qa
optimization_date: '2025-12-11T11:44:16.119752'
original_url: https://thebuildingcoder.typepad.com/blog/1479_roomedit3d_running.html
post_number: '1479'
reading_time_minutes: 8
series: general
slug: roomedit3d_running
source_file: 1479_roomedit3d_running.md
tags:
- csharp
- elements
- family
- parameters
- revit-api
- rooms
- sheets
- transactions
- views
- walls
title: The Building Coder
word_count: 1625
---

### Roomedit3dv3 Up and Running with Demo Recording
I completed the [roomedit3dv3](https://github.com/Autodesk-Forge/forge-boilers.nodejs/tree/roomedit3d) project:
- [Enhancements to the Forge Node.js boilerplate code](#2)
- [RoomEditorApp Revit add-in update](#3)
- [Roomedit3dv3 demo recording](#4)
- [Samples connecting desktop and cloud](#5)
Before I get to that, I should also mention that
the [Forge webinar series](http://autodeskforge.devpost.com/details/webinars) session 5
is being held today, by Albert Szilvasy, on the Design Automation API.
You can register by [clicking here](https://global.gotowebinar.com/join/134416283).
#### Forge Webinar Series 5 – Design Automation API
Recordings, presentations and support material of the past sessions are available for viewing and download:
- September 20 – [Introduction to Autodesk Forge and the Autodesk App Store](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-autodesk-forge-and-the-autodesk-app-store.html)
- September 22 – [Introduction to OAuth and Data Management API](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-oauth-and-data-management-api.html)
– on [OAuth](https://developer.autodesk.com/en/docs/oauth/v2/overview)
and [Data Management API](https://developer.autodesk.com/en/docs/data/v2/overview), providing token-based authentication, authorization and a unified and consistent way to access data across A360, Fusion 360, and the Object Storage Service.
- September 27 – [Introduction to Model Derivative API](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-model-derivative-api.html)
– on the [Model Derivative API](https://developer.autodesk.com/en/docs/model-derivative/v2/overview) that enables users to represent and share their designs in different formats and extract metadata.
- September 29 – [Introduction to Viewer](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-viewer-api.html)
– the [Viewer](https://developer.autodesk.com/en/docs/viewer/v2/overview), formerly part of the 'View and Data API', is a WebGL-based JavaScript library for 3D and 2D model rendering of CAD models from seed files, e.g., [AutoCAD](http://www.autodesk.com/products/autocad/overview), [Fusion 360](http://www.autodesk.com/products/fusion-360/overview), [Revit](http://www.autodesk.com/products/revit-family/overview) and many other formats.
Upcoming sessions continue during the remainder of
the [Autodesk App Store Forge and Fusion 360 Hackathon](http://autodeskforge.devpost.com) until the end of October:
- October 4 – [Design Automation API](https://developer.autodesk.com/en/docs/design-automation/v2/overview) – formerly known as 'AutoCAD I/O', run scripts on design files.
- October 6 – [BIM360](https://developer.autodesk.com/en/docs/bim360/v1/overview) – develop apps that integrate with BIM 360 to extend its capabilities in the construction ecosystem.
- October 11 – [Fusion 360 Client API](http://help.autodesk.com/view/NINVFUS/ENU/?guid=GUID-A92A4B10-3781-4925-94C6-47DA85A4F65A) – an integrated CAD, CAM, and CAE tool for product development, built for the new ways products are designed and made.
- October 13 – Q&A on all APIs.
- October 20 – Q&A on all APIs.
- October 27 – Submitting a web service app to Autodesk App store.
Quick access links:
- For API keys, go to [developer.autodesk.com](https://developer.autodesk.com)
- For code samples, go to [github.com/Developer-Autodesk](https://github.com/Developer-Autodesk)
Feel free to contact us at [forgehackathon@autodesk.com](mailto:forgehackathon@autodesk.com) at any time with any questions.
Back to the room editor:
![Roomedit3dv3 Forge extension in action](img/roomedit3dv3_broadcast.png)
#### Enhancements to the Forge Node.js Boilerplate Code
Roomedit3dv3 builds on [Philippe Leefsma](http://twitter.com/F3lipek)'s
[Forge node.js boilerplate samples](https://github.com/Autodesk-Forge/forge-boilers.nodejs),
adding a few simple enhancements in order to demonstrate interactive movement of BIM element instances in the Forge viewer and updating the Revit model accordingly in real time via a `socket.io` broadcast.
The additional enhancements include:
- [Adding a viewer `transform` extension](http://thebuildingcoder.typepad.com/blog/2016/09/warning-swallower-and-roomedit3d-viewer-extension.html#3).
- [Retrieving and broadcasting the affected element `UniqueId` and translation](http://thebuildingcoder.typepad.com/blog/2016/10/retrieving-and-broadcasting-the-roomedit3dv3-translation.html) via `socket.io`.
I created a new Forge app for the production version, set up its `PROD` environment and deployed to [Heroku](http://heroku.com) with no major issues.
Everything went well, as you can see below in my description of the [Revit add-in adaptation](#3) and [test run](#4).
Here is are all the nitty-gritty details on choosing a suitable starting point and adapting the boilerplate code for my needs:
- [Roomedit3d Update for Connecting Desktop and Forge](http://thebuildingcoder.typepad.com/blog/2016/09/roomedit3d-update-for-connecting-desktop-and-forge.html)
- [More Roomedit3dv3 Starting Points](http://thebuildingcoder.typepad.com/blog/2016/09/forge-webinar-series-and-more-roomedit-starting-point-samples.html#2)
- [The Birth of Roomedit3dv3](http://thebuildingcoder.typepad.com/blog/2016/09/the-birth-of-roomedit3dv3-and-forge-webinar-series.html#2)
- [REST vs WebSocket and Roomedit3dv3 Broadcast Architecture](http://thebuildingcoder.typepad.com/blog/2016/09/roomedit3d-broadcast-teigha-bim-and-forge-webinar-3.html#2)
- [Roomedit3dv3 Transform Viewer Extension](http://thebuildingcoder.typepad.com/blog/2016/09/warning-swallower-and-roomedit3d-viewer-extension.html#3)
- [Retrieving and Broadcasting the Roomedit3dv3 Translation](http://thebuildingcoder.typepad.com/blog/2016/10/retrieving-and-broadcasting-the-roomedit3dv3-translation.html)
Now that the web server part is complete, let's test it from Revit:
#### RoomEditorApp Revit Add-in Update
The `socket.io` broadcast is picked up by the [Roomedit3dApp](https://github.com/jeremytammik/Roomedit3dApp) Revit add-in and applied to the Revit model just like in previous versions.
All I had to do was update the `socket.io` URL.
I also added a check for the correct Revit project document and a valid element `UniqueId`, since the viewer extension might be running in any number of instances for various users at the same time, and no information is currently added to the socket.io broadcast to tell which Revit model is being edited by who.
Therefore, in the current implementation, you might be bombarded with millions of transform messages from all the thousands of different users playing with this simultaneously worldwide.
Thus the importance of checking whether the notification you just received really does apply and can be applied to your current document.
Here is the new `BimUpdater.Execute` method implementation that hopefully takes care of that:
```csharp
///
/// Execute method invoked by Revit via the
/// external event as a reaction to a call
/// to its Raise method.
/// summary>
public void Execute( UIApplication a )
{
Debug.Assert( 0 < _queue.Count,
"why are we here with nothing to do?" );
Document doc = a.ActiveUIDocument.Document;
// Ensure that the unique id refers to a valid
// element from the current doucument. If not,
// no need to start a transaction.
string uid = _queue.Peek().Item1;
Element e = doc.GetElement( uid );
if( null != e )
{
using( Transaction t = new Transaction( doc ) )
{
t.Start( GetName() );
while( 0 < _queue.Count )
{
Tuple task = _queue.Dequeue();
Debug.Print( "Translating {0} by {1}",
task.Item1, Util.PointString( task.Item2 ) );
e = doc.GetElement( task.Item1 );
if( null != e )
{
ElementTransformUtils.MoveElement(
doc, e.Id, task.Item2 );
}
}
t.Commit();
}
}
}
```
I tagged the
new [Roomedit3dApp](https://github.com/jeremytammik/Roomedit3dApp)
version [release 2017.0.0.6](https://github.com/jeremytammik/Roomedit3dApp/releases/tag/2017.0.0.6).
#### Roomedit3dv3 Demo Recording
With everything up and running smoothly, I created
a [ten-minute demo recording](https://youtu.be/wNtVBcbfhSw) on
the [Roomedit3dv3 Revit BIM Autodesk Forge Viewer Extension](https://youtu.be/wNtVBcbfhSw):

To summarise, it demonstrates real-time round-trip editing of Revit BIM via the Autodesk Forge Viewer using
[Roomedit3dv3](https://github.com/Autodesk-Forge/forge-boilers.nodejs/tree/roomedit3d), which implements
a Forge Viewer extension to move building elements and update the Revit BIM in real-time using `socket.io`.
You can easily deploy your own version of it to Heroku with one simple button click, cf.
the [instructions in the GitHub repository](https://github.com/Autodesk-Forge/forge-boilers.nodejs/tree/roomedit3d#prerequisites-and-sample-setup),
or test run my own web personal server instance directly at [roomedit3dv3.herokuapp.com](https://roomedit3dv3.herokuapp.com).
#### Further Samples Connecting Desktop and Cloud
I continue testing and documenting the other samples connecting the desktop and the cloud for
the [RTC Revit Technology Conference Europe](http://www.rtcevents.com/rtc2016eur) in Porto,
each consisting of two components, a C# .NET Revit API desktop add-in and a web server:
- [RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp) and
the [roomeditdb](https://github.com/jeremytammik/roomedit)
[CouchDB](https://couchdb.apache.org)
database and web server demonstrating real-time round-trip graphical editing of furniture family instance location and rotation plus textual editing of element properties in a simplified
2D [SVG](https://www.w3.org/Graphics/SVG/) representation of the 3D BIM.
- [FireRatingCloud](https://github.com/jeremytammik/FireRatingCloud) and
the [fireratingdb](https://github.com/jeremytammik/firerating)
[node.js](https://nodejs.org)
[MongoDB](https://www.mongodb.com) web server demonstrating real-time round-trip editing of Revit element shared parameter values stored in
a globally accessible [mongolab](http://mongolab.com)-hosted db.
- [Roomedit3dApp](https://github.com/jeremytammik/Roomedit3dApp) and
the first [roomedit3d](https://github.com/jeremytammik/roomedit3d) Forge Viewer extension demonstrating translation of BIM elements in the viewer and updating the Revit model in real time via a 'socket.io' broadcast.
- The sample discussed above, adding the option to select any Revit model hosted
on [A360](https://a360.autodesk.com), again using
the [Roomedit3dApp](https://github.com/jeremytammik/Roomedit3dApp) Revit add-in working with the
new [roomedit3dv3](https://github.com/Autodesk-Forge/forge-boilers.nodejs/tree/roomedit3d)
[Autodesk Forge](https://forge.autodesk.com)
[Viewer](https://developer.autodesk.com/en/docs/viewer/v2/overview) extension
to demonstrate translation of BIM element instances in the viewer and updating the Revit model in real time via a `socket.io` broadcast.
All is still looking good so far.