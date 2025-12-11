---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.0
content_type: qa
optimization_date: '2025-12-11T11:44:16.865596'
original_url: https://thebuildingcoder.typepad.com/blog/1840_camera_fov.html
post_number: '1840'
reading_time_minutes: 3
series: general
slug: camera_fov
source_file: 1840_camera_fov.md
tags:
- elements
- revit-api
- sheets
- views
title: Camera Fov
word_count: 643
---

### Revit Camera FOV, Forge Partner Talks and Jobs
Today we resuscitate a five-year old Revit API answer, still as fresh and useful as ever, followed by Forge and job opportunities truly fresh off the presses:
- [Determining the Revit Camera FOV](#2)
- [Forge Partner Talks](#3)
- [Jobs at Autodesk](#4)
#### Determining the Revit Camera FOV
Many people have asked how you can retrieve the Revit camera field of view or FOV.
As Valerii Nozdrenkov points out in
his [comment](https://thebuildingcoder.typepad.com/blog/2019/06/revit-camera-settings-project-plasma-da4r-and-ai.html#comment-4891620499)
on [mapping the Revit camera settings to the Forge viewer](https://thebuildingcoder.typepad.com/blog/2019/06/revit-camera-settings-project-plasma-da4r-and-ai.html),
The Building Coder already published a solution suggested by Arnošt Löbel reading the required data from
the [custom exporter `GetCameraInfo`](https://thebuildingcoder.typepad.com/blog/2014/09/custom-exporter-getcamerainfo.html):
> When a view is processed and run through a custom exporter context, its properties are used to populate a `ViewNode` instance.
> One of its methods is `GetCameraInfo`, which provides information that ought to cover everything you need to know about the view's camera.

```
  public RenderNodeAction OnViewBegin(ViewNode node)
  {
    CameraInfo cameraInfo = node.GetCameraInfo();

    var view = document.GetElement(node.ViewId);
    return RenderNodeAction.Proceed;
  }
```

It seems worthwhile to reiterate this, since the question keeps popping up...
![Camera angle of view](img/camera_fov_lens_angle_of_view.png "Camera angle of view")
#### Forge Partner Talks
The virtual Forge accelerators are leading to a large number of technical webinars and zoom meetings with great attendance and participation.
It is time to also hold some 'business creation' meetings with Forge partners.
Here are some upcoming webinars in this area that might be of interest to you:
- [Forge Partner Talks: Digital Twins](https://autodesk.zoom.us/webinar/register/7415875742427/WN_UiMEtQNiTFiPH8T_ekwk4w) (May 6 @ 7am PST/4pm CET)
– Digital twins were one of the top ten tech trends last year, according to Gartner and Deloitte. Come and hear from Forge partners who have built or leveraged successful digital twin solutions and see how you can too.
- [Forge Partner Talks: AR/VR](https://autodesk.zoom.us/webinar/register/6515877525566/WN_W6abt1RSR5K3V44HV5CtXQ) (May 13 @ 7am PST/4pm CET)
– See how leading Autodesk partners are bringing Forge-powered AR/VR solutions to the design and make space and learn how these solutions can benefit you.
- [Forge Partner Talks: Configurators](https://autodesk.zoom.us/webinar/register/8415877525897/WN_01GeR_H9RKOGTOg67Unagg) (May 20 @ 7am PST/4pm CET)
– Hear from Forge partners who have increased their customer satisfaction by creating Forge-powered configurators and see how providing similar customization tools to your customers can help you maintain your competitive edge.
- [Forge Partner Talks: Dashboards and Insights](https://autodesk.zoom.us/webinar/register/7515877526195/WN_vDxsTlF4QgS3s_8qIXYEXg) (May 27@ 7am PST/4pm CET)
– It's all about the data! Discover some of the innovative dashboards our top Forge partners have built, and see how these types of solutions can help you better visualize your data, draw insights, and be more profitable.
Looking forward to having you there!
#### Jobs at Autodesk
Finally... Would you like to work with the Forge development team?
We hope you’re all staying safe and healthy in your homes.
In spite of the current situation, Autodesk is still actively hiring.
Forge continues to search for top talent as our global workforce works remotely.
Would you be a good fit?
Here are a few roles highlighted in April and May:
- [20WD39053 – Senior Technical Writer, Developer Content – This role can be located anywhere](https://rolp.co/68d5i)
- [20WD39597 – Senior Software Engineer, Cloud Service – Shanghai](https://rolp.co/IUdfi)
- [19WD36553 – Principal Engineer – UK](https://rolp.co/Pc5Li)
- [20WD38369 – Software Architect – Singapore](https://rolp.co/JWHti)
- [20WD39627 – Senior Vendor Manager – San Francisco](https://rolp.co/RUAfi)
- [20WD38632 – Engineering Manager – Portland](https://rolp.co/Dmtei)
- [20WD38618 – Software Development Manager – Novi](https://rolp.co/SrG6i)