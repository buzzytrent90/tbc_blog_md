---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.6
content_type: qa
optimization_date: '2025-12-11T11:44:14.596835'
original_url: https://thebuildingcoder.typepad.com/blog/0790_getting_going_cloud.html
post_number: 0790
reading_time_minutes: 10
series: general
slug: getting_going_cloud
source_file: 0790_getting_going_cloud.htm
tags:
- csharp
- elements
- geometry
- levels
- parameters
- revit-api
- views
- windows
title: Getting Going with the Cloud
word_count: 1978
---

### Getting Going with the Cloud

I am now the proud owner of an Android tablet and happily thinking about things to do with it.
The first idea that comes to mind is working through something similar to Adam's
[Revit model viewer for iOS](http://adndevblog.typepad.com/aec/2012/06/revit-model-viewer-for-ios-part-1.html) that
Jim Quanci showed at the
[AEC DevCamp](http://thebuildingcoder.typepad.com/blog/2012/06/devcamp-day-one.html).

Before getting into that, though, I should address some of the issues brought up by Senthil in
[his](http://thebuildingcoder.typepad.com/blog/2009/08/serviceoriented-architecture.html?cid=6a00e553e1689788330168ebd5548e970c#comment-6a00e553e1689788330168ebd5548e970c)
[comment](http://thebuildingcoder.typepad.com/blog/2009/08/serviceoriented-architecture.html?cid=6a00e553e168978833016767cacd01970b#comment-6a00e553e168978833016767cacd01970b)
on the discussion on
[service-oriented architecture](http://thebuildingcoder.typepad.com/blog/2009/08/serviceoriented-architecture.html),
asking for information about Revit cloud development, e.g. a step by step process for cloud beginners.
Senthil also mentions the
[cloud computing Revit demo](http://download.autodesk.com/media/adn/cloudcomputing/3_DevTV_Cloud_Computing_Revit_Demo/DevTV_Cloud_Computing_Revit_Demo.html) on the
[ADN cloud computing page](http://usa.autodesk.com/adsk/servlet/item?siteID=123112&id=17136545),
its add-in manifest, assembly DLL file location, how to deploy it, how to interact with Revit through web services, and developing a service application in general.

Here are a couple of starting points for addressing these issues:

- [DevCamp cloud and mobile track](#2)- [The Apollonian gasket cloud service](#3)- [The AEC cloud demo](#4)- [Revit model viewer](#5)

#### DevCamp Cloud and Mobile Track

All of these topics are addressed in depth by Gopinath Taget's cloud and mobile presentation, both at Autodesk University last year and most recently at the AEC and Manufacturing DevCamps earlier this month, each of which boasted a separate track on cloud and mobile technologies.
Gopi's four sessions at the AEC DevCamp covered:

- **An Overview of Cloud Computing:**
  Learn what cloud computing is all about, what kind of applications can be written for and run on the cloud, when it is suitable to use the cloud, when it is not. Learn about the popular commercial cloud service providers including Amazon Web Services (AWS), Microsoft Azure and Google App Engine and how to use them. Learn the similarities and differences between the cloud services they provide, the advantages of using one over the others and the coverage and sophistication of the APIs provided to use their cloud services. The class includes demonstrations and code review of sample cloud applications for Autodesk Revit, AutoCAD Civil 3D, AutoCAD and AutoCAD WS.- **State of the Art of Autodesk Cloud and Mobile Apps:**
    Learn about current Autodesk cloud services, their capabilities and APIs. We will talk about and demonstrate AutoCAD WS, Autodesk 360, and Autodesk Photofly web services in depth. Demonstrations and instruction will be based on Microsoft Windows, Apple's iOS and Google's Android. The class will end on an exploration where these web services are going and - at a high level - where Autodesks web services are headed.- **Introduction to Mobile app development – Apple's iOS:**
      Learn about programming on iOS devices. Learn where you need to go and what you need to do to start programming on iOS. Learn about the SDKs important for graphics intensive software development including WebGL and OpenGL ES and how to get started working with them. This class includes a detailed "start to finish" look at development of a simple iOS mobile App with a basic user interface.- **Introduction to Mobile app development – Google's Android:**
        Learn about programming on Android devices. Learn where you need to go and what you need to do to start programming on Android. Learn about the SDKs that would be important to a CAD developer like WebGL and OpenGL ES and how to get started on them. This class will also demonstrate creating a simple Android mobile App with simple user interface from start to finish.

Gopi's presentations and samples are currently publicly available from Buzzsaw together with all the rest of the AEC and Manufacturing
[DevCamp 2012 material](http://thebuildingcoder.typepad.com/blog/2012/06/aec-devcamp-2012-material.html).

#### The Apollonian Gasket Cloud Service

Probably the greatest and deepest exploration of using cloud services over a wide variety of technologies and devices was created by Kean Walmsley.
He explores making use of just about all imaginable combinations of cloud service providers and clients.
The
[Apollonian gasket cloud & mobile series summary](http://through-the-interface.typepad.com/through_the_interface/2012/06/cloud-mobile-series-summary.html) boasts
the following impressive table of contents:

##### Creating the core desktop functionality

- [Circle packing in AutoCAD: creating an Apollonian gasket using F# – Part 1](http://through-the-interface.typepad.com/through_the_interface/2012/02/circle-packing-in-autocad-creating-an-apollonian-gasket-using-f-part-1.html)
- [Circle packing in AutoCAD: creating an Apollonian gasket using F# – Part 2](http://through-the-interface.typepad.com/through_the_interface/2012/02/circle-packing-in-autocad-creating-an-apollonian-gasket-using-f-part-2.html)
- [Sphere packing in AutoCAD: creating an Apollonian packing using F# – Part 1](http://through-the-interface.typepad.com/through_the_interface/2012/02/sphere-packing-in-autocad-creating-an-apollonian-packing-using-f-part-1.html)
- [Sphere packing in AutoCAD: creating an Apollonian packing using F# – Part 2](http://through-the-interface.typepad.com/through_the_interface/2012/02/sphere-packing-in-autocad-creating-an-apollonian-packing-using-f-part-2.html)

##### Moving it to the cloud

- [Moving to the Cloud](http://through-the-interface.typepad.com/through_the_interface/2012/03/moving-to-the-cloud.html)
- [Exposing a RESTful web service for use inside AutoCAD using the ASP.NET Web API – Part 1](http://through-the-interface.typepad.com/through_the_interface/2012/04/exposing-a-restful-web-service-for-use-inside-autocad-using-the-aspnet-web-api-part-1.html)
- [Exposing a RESTful web service for use inside AutoCAD using the ASP.NET Web API – Part 2](http://through-the-interface.typepad.com/through_the_interface/2012/04/exposing-a-restful-web-service-for-use-inside-autocad-using-the-aspnet-web-api-part-2.html)
- [Architecting for the Cloud](http://through-the-interface.typepad.com/through_the_interface/2012/04/architecting-for-the-cloud.html)
- [Consuming data from a RESTful web-service inside AutoCAD using .NET](http://through-the-interface.typepad.com/through_the_interface/2012/04/consuming-data-from-a-restful-web-service-inside-autocad-using-net.html)
- [Hosting our ASP.NET Web API project on Windows Azure – Part 1](http://through-the-interface.typepad.com/through_the_interface/2012/04/hosting-our-aspnet-web-api-project-on-windows-azure-part-1.html)
- [Hosting our ASP.NET Web API project on Windows Azure – Part 2](http://through-the-interface.typepad.com/through_the_interface/2012/04/hosting-our-aspnet-web-api-project-on-windows-azure-part-2.html)
- [Using Windows Azure Caching with our ASP.NET Web API project](http://through-the-interface.typepad.com/through_the_interface/2012/04/using-windows-azure-caching-with-our-aspnet-web-api-project.html)

##### Using the web-service from various clients

- AutoCAD
  - [Calling a cloud-based web-service from AutoCAD using .NET](http://through-the-interface.typepad.com/through_the_interface/2012/04/calling-a-cloud-based-web-service-from-autocad-using-net.html)
- Unity3D
  - [Calling a web-service from a Unity3D scene](http://through-the-interface.typepad.com/through_the_interface/2012/04/calling-a-web-service-from-a-unity3d-scene.html)
- Android
  - [Creating a 3D viewer for our Apollonian service using Android – Part 1](http://through-the-interface.typepad.com/through_the_interface/2012/04/creating-a-3d-viewer-for-our-apollonian-service-using-android-part-1.html)
  - [Creating a 3D viewer for our Apollonian service using Android – Part 2](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-android-part-2.html)
  - [Creating a 3D viewer for our Apollonian service using Android – Part 3](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-android-part-3.html)
- iOS
  - [Creating a 3D viewer for our Apollonian service using iOS – Part 1](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-ios-part-1.html)
  - [Creating a 3D viewer for our Apollonian service using iOS – Part 2](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-ios-part-2.html)
  - [Creating a 3D viewer for our Apollonian service using iOS – Part 3](http://through-the-interface.typepad.com/through_the_interface/2012/06/creating-a-3d-viewer-for-our-apollonian-service-using-ios-part-3-1.html)
- HTML5/WebGL
  - [Creating a 3D viewer for our Apollonian service using HTML5 – Part 1](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-html5.html)
  - [Creating a 3D viewer for our Apollonian service using HTML5 – Part 2](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-html5-part-2.html)
  - [Creating a 3D viewer for our Apollonian service using HTML5 – Part 3](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-html5-part-3.html)
- WinRT
  - [Creating a 3D viewer for our Apollonian service using WinRT – Part 1](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-winrt-part-1.html)
  - [Creating a 3D viewer for our Apollonian service using WinRT – Part 2](http://through-the-interface.typepad.com/through_the_interface/2012/05/creating-a-3d-viewer-for-our-apollonian-service-using-winrt-part-2.html)

This really is a huge wealth of material and should provide ample words of wisdom for any brave seeker venturing out on these now no longer unmapped paths.

#### AEC Cloud Demo

Returning to my own much more modest efforts in this realm so far based on the Revit API, Senthil mentions the
[AEC cloud demo](http://download.autodesk.com/media/adn/cloudcomputing/3_DevTV_Cloud_Computing_Revit_Demo/DevTV_Cloud_Computing_Revit_Demo.html) on the
[ADN cloud computing page](http://usa.autodesk.com/adsk/servlet/item?siteID=123112&id=17136545).
That is an eleven minute recording of a very simple add-in running on Revit 2012, originally created for the DevDays conferences in the end of 2010.

I recently migrated the sample add-in to Revit 2013.
Here is
[AecMatInfoClientRevit2013.zip](zip/AecMatInfoClientRevit2013.zip) containing
its entire source code, Visual Studio solution and add-in manifest.

As explained in the recording, it simply defines two commands to read data from certain Revit element parameters and store them in a simple cloud-hosted database, and vice versa to read data from the database and populate it back into the Revit parameters again.

#### Revit Viewer via OBJ

As said, the next thing I would like to look at is a Revit model viewer.

Instead of using Adam's approach via a custom geometry file format, I thought I might make use of the
[Wavefront OBJ](http://en.wikipedia.org/wiki/Wavefront_.obj_file) file
format, which seems to be pretty standard and compact.

My current tentative outline looks like this:

1. Describe Adam's Revit cloud and mobile demo (well, Adam is doing this himself):
   1. [RVT add-in](http://adndevblog.typepad.com/aec/2012/06/revit-model-viewer-for-ios-part-1.html) exporting triangulated geometry faces via custom ASCII file format, uploading to cloud- Mobile device accessing cloud data, custom iOS viewer reading and displaying 3D view of custom format- Implement a RVT add-in that exports OBJ file format instead
     1. Standard format, can be used ubiquitously- Optimise that file format to significantly reduce file size and increase speed and efficiency- View OBJ file format on mobile
       1. Using standard viewer- Implement custom Android viewer

It will be interesting to see how I do during the next few days... especially for me :-)