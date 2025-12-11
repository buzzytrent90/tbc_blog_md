---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 8.4
content_type: qa
optimization_date: '2025-12-11T11:44:16.135566'
original_url: https://thebuildingcoder.typepad.com/blog/1486_webgl_forge_intro.html
post_number: '1486'
reading_time_minutes: 10
series: general
slug: webgl_forge_intro
source_file: 1486_webgl_forge_intro.md
tags:
- family
- filtering
- parameters
- references
- revit-api
- rooms
- schedules
- sheets
- views
title: The Building Coder
word_count: 2089
---

### Forge Formats, Webinars and Fusion 360 Client API
I updated
the [WebGL and Forge introduction for BIM programming](http://jeremytammik.github.io/forge_bim_programming) and
its [GitHub source](https://github.com/jeremytammik/forge_bim_programming) for
the presentations in the coming days at
the [RTC Revit Technology Conference Europe](http://www.rtcevents.com/rtc2016eur) and
the [ISEPBIM](https://www.facebook.com/ISEPBIM) Forge and BIM workshops at [ISEP](http://www.isep.ipp.pt),
the [Instituto Superior de Engenharia do Porto](http://www.isep.ipp.pt),
implemented two little `curl` wrapper scripts to help me list the supported file formats, explored why they changed and updated the hackathon webinar overview.
Before getting to that, though, I'll also highlight a helpful little note by Christian on how to access the 'Symbolic Representation' setting:
- ['Symbolic Representation' setting parameter](#1)
- [Forge intro for BIM programming](#2)
- [`cURL` wrapper scripts to list Forge file formats](#3)
- [Updated Forge file formats](#4)
- [Forge hackathon webinar series and Fusion 360 Client API recording](#5)
#### 'Symbolic Representation' Setting Parameter
Christian shared how to access the 'Symbolic Representation' setting in a family definition in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) thread
on [symbolic representation - parameter](http://forums.autodesk.com/t5/revit-api-forum/symbolic-representation-parameter/m-p/6623135):
> just wanted to share some code as I "lost" one hour of my life to look for it. If you want to access the setting for "Symbolic Representation" inside a family document, here we go:
```csharp
Parameter param = famdoc.OwnerFamily.get_Parameter(
BuiltInParameter.FAMILY_SYMBOLIC_REP );
if( param != null )
{
// 0 - From Family
// 1 - From Project Setting
param.Set( 1 );
}
```
Thank you very much for sharing that, Christian!
#### Forge Intro for BIM Programming
[Forge for BIM Programming](https://github.com/jeremytammik/forge_bim_programming) is
a [reveal.js](https://github.com/hakimel/reveal.js) presentation
on [WebGL](https://www.khronos.org/webgl)
and [Autodesk Forge platform](https://developer.autodesk.com) introduction.
It was first used for
the [BIM Programming](http://www.bimprogramming.com) conference
and workshops in Madrid, January 2016, and now updated for
the [RTC Revit Technology Conference Europe](http://www.rtcevents.com/rtc2016eur) and
the [ISEPBIM](https://www.facebook.com/ISEPBIM) Forge and BIM workshops at [ISEP](http://www.isep.ipp.pt),
the [Instituto Superior de Engenharia do Porto](http://www.isep.ipp.pt) in Porto, October 2016.
For more information on Forge, please refer
to the overview at [forge.autodesk.com](https://forge.autodesk.com)
and all the technical details, samples and further documentation
at [developer.autodesk.com](https://developer.autodesk.com).
To view this presentation online, visit [jeremytammik.github.io/forge_bim_programming](http://jeremytammik.github.io/forge_bim_programming).
![Forge platform diagram](img/forge_platform_diagram.png)
#### `cURL` Wrapper Scripts to List Forge File Formats
One of the slides in the presentation above lists the file formats currently supported by the Forge platform.
This list can be obtained by a REST call to
the [Forge](https://developer.autodesk.com/)
[Model Derivative API](https://developer.autodesk.com/en/docs/model-derivative/v2/overview/)
[GET formats](https://developer.autodesk.com/en/docs/model-derivative/v2/reference/http/formats-GET/) endpoint:
I had an temporarily insurmountable problem with Typepad crashing at this point... it was resolved by escaping the character `c` using `c` in all calls to the `curl` command in the following scripts, i.e., replacing the string `curl` by `curl`. Weird, probably protects against some potential hack, and now solved...
You can look at what I actually intend to publish in
the [tbc GitHub repository](https://github.com/jeremytammik/tbc),
where this blog posts is mirrored
as [tbc/a/1486_webgl_forge_intro.html](http://jeremytammik.github.io/tbc/a/1486_webgl_forge_intro.html).
Sorry, I am unable to add the scripts inline to this typepad post.
Here they are as links instead:
- [forgeauth](zip/forgeauth)
- [forgeformats](zip/forgeformats)
OK, back to normal now, problem resolved...

```
curl -X 'GET' -H 'Authorization: Bearer ztcaB2R0f92bsV6iV0bSDgwmSVaW' -v 'https://developer.api.autodesk.com/modelderivative/v2/designdata/formats'
```

We need to supply a valid access token, though, instead of the placeholder listed above.
This in turn is obtained by a call to
the [authentication](https://developer.autodesk.com/en/docs/oauth/v2/overview/)
[POST authenticate](https://developer.autodesk.com/en/docs/oauth/v2/reference/http/authenticate-POST/) endpoint,
specifying our client id and secret.
To easily complete this two-step process from the command line, I implemented two Unix shell scripts, `forgeauth` and `forgeformats`.
The former retrieves my client id and secret from two environment variables and calls the `authenticate` endpoint like this:

```
# !/bin/bash

curl -v 'https://developer.api.autodesk.com/authentication/v1/authenticate' -X 'POST' -H 'Content-Type: application/x-www-form-urlencoded' -d "client_id=$ROOMEDIT3DV3_PROD_CONSUMER_KEY&client_secret=$ROOMEDIT3DV3_PROD_CONSUMER_SECRET&grant_type=client_credentials&scope=data:read"

echo "Now you might want to export FORGE_ACCESS_TOKEN=... for consumption by forgeformats"
```

This returns the following JSON formatted access token data:

```
{"access_token":"XzVxWJoVbNWcnJeKJPRnkYjp1tgt","token_type":"Bearer","expires_in":86399}
```

As the script suggests, I store the transient access token in another environment variable `FORGE_ACCESS_TOKEN`:

```
$ export FORGE_ACCESS_TOKEN=XzVxWJoVbNWcnJeKJPRnkYjp1tgt
```

From there, it is retrieved by the second script, `forgeformats`, which feeds it into the second endpoint like this:

```
# !/bin/bash
curl -X 'GET' -H "Authorization: Bearer $FORGE_ACCESS_TOKEN" -v 'https://developer.api.autodesk.com/modelderivative/v2/designdata/formats'
```

Right now, that returns the following list:

```
{"formats":{"svf":["3dm","3ds","asm","catpart","catproduct","cgr","collaboration","dae","dgn","dlv3","dwf","dwfx","dwg","dwt","dxf","exp","f3d","fbx","g","gbxml","iam","idw","ifc","ige","iges","igs","ipt","jt","model","neu","nwc","nwd","obj","pdf","prt","rcp","rvt","sab","sat","session","skp","sldasm","sldprt","smb","smt","ste","step","stl","stla","stlb","stp","stpz","wire","x_b","x_t","xas","xpr","zip","prt\.\d+$","neu\.\d+$","asm\.\d+$"],"thumbnail":["3dm","3ds","asm","catpart","catproduct","cgr","collaboration","dae","dgn","dlv3","dwf","dwfx","dwg","dwt","dxf","exp","f3d","fbx","g","gbxml","iam","idw","ifc","ige","iges","igs","ipt","jt","model","neu","nwc","nwd","obj","pdf","prt","rcp","rvt","sab","sat","session","skp","sldasm","sldprt","smb","smt","ste","step","stl","stla","stlb","stp","stpz","wire","x_b","x_t","xas","xpr","zip","prt\.\d+$","neu\.\d+$","asm\.\d+$"],"stl":["f3d","fbx","iam","ipt","wire"],"step":["f3d","fbx","iam","ipt","wire"],"iges":["f3d","fbx","iam","ipt","wire"],"obj":["f3d","fbx","iam","ipt","step","stp","stpz","wire"]}}
```

#### Updated Forge file formats
Here is the list of extensions currently returned by call to the `formats` endpoint shown above, cleaned up and alphabetically sorted:
- 3dm, 3ds, asm, asm, catpart, catproduct, cgr, collaboration, dae, dgn, dlv3, dwf, dwfx, dwg, dwt, dxf, exp, f3d, fbx, g, gbxml, iam, idw, ifc, ige, iges, igs, ipt, jt, model, neu, neu, nwc, nwd, obj, pdf, prt, prt, rcp, rvt, sab, sat, session, skp, sldasm, sldprt, smb, smt, ste, step, stl, stla, stlb, stp, stpz, wire, x_b, x_t, xas, xpr, zip
I compared that with [the previous list that I generated in January](http://the3dwebcoder.typepad.com/blog/2015/04/displaying-2d-graphics-via-a-node-server.html#3.1):
- 3dm, 3ds, asm, bmp, cam360, catpart, catproduct, cgr, csv, dae, dlv3, doc, docx, dwf, dwfx, dwg, dwt, exp, f3d, fbx, g, gbxml, iam, ifc, ige, iges, igs, ipt, jpeg, jpg, jt, model, neu, nwc, nwd, obj, pdf, png, pps, ppt, pptx, prt, rcp, rtf, rvt, sab, sat, session, sim, sim360, skp, sldasm, sldprt, smb, smt, ste, step, stl, stla, stlb, stp, tif, tiff, txt, wire, x_b, x_t, xas, xls, xlsx, xpr, zip
As you can see, support for quite a number of new formats has been added.
A few have also been removed, such as BMP and TIFF.
I asked for an explanation for that, and learned the following:
Here are the ones that seem to have been added:
- collaboration, dgn, dxf, idw, stpz
We assume it was easy (and made sense) to extend to more CAD-related formats such as `dgn`, `dxf` and `idw`.
`Stpz` is probably just compressed STP, so that was presumably a no-brainer, too.
The following have been removed:
- bmp, cam360, csv, doc, docx, jpeg, jpg, png, pps, ppt, pptx, rtf, sim, sim360, tif, tiff, txt, xls, xlsx
Aside from `cam360`, `sim` and `sim360`, these are all Office or image formats.
Some were not actually supported by the Model Derivative service but are supported in the A360 product and its viewers.
This looks like good progress, to me.
Many thanks to Adam, Kean and Lee for their analysis, suggestions and clarifications!
#### Forge Hackathon Webinar Series and Fusion 360 Client API Recording
The [Forge webinar series](http://autodeskforge.devpost.com/details/webinars) is nearing its end.
Here are the recordings and documentation pointers for the topics covered so far:
- September 20
– [Introduction to Autodesk Forge and the Autodesk App Store](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-autodesk-forge-and-the-autodesk-app-store.html)
- September 22
– [Introduction to OAuth and Data Management API](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-oauth-and-data-management-api.html)
– on [OAuth](https://developer.autodesk.com/en/docs/oauth/v2/overview)
and [Data Management API](https://developer.autodesk.com/en/docs/data/v2/overview), providing token-based authentication, authorization and a unified and consistent way to access data across A360, Fusion 360, and the Object Storage Service.
- September 27
– [Introduction to Model Derivative API](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-model-derivative-api.html)
– on the [Model Derivative API](https://developer.autodesk.com/en/docs/model-derivative/v2/overview) that enables users to represent and share their designs in different formats and extract metadata.
- September 29
– [Introduction to Viewer](http://adndevblog.typepad.com/cloud_and_mobile/2016/09/introduction-to-viewer-api.html)
– the [Viewer](https://developer.autodesk.com/en/docs/viewer/v2/overview), formerly part of the 'View and Data API', is a WebGL-based JavaScript library for 3D and 2D model rendering of CAD models from seed files, e.g., [AutoCAD](http://www.autodesk.com/products/autocad/overview), [Fusion 360](http://www.autodesk.com/products/fusion-360/overview), [Revit](http://www.autodesk.com/products/revit-family/overview) and many other formats.
- October 4
– [Introduction to Design Automation](http://adndevblog.typepad.com/cloud_and_mobile/2016/10/introduction-to-design-automation.html)
– the [Design Automation API](https://developer.autodesk.com/en/docs/design-automation/v2/overview), formerly known as 'AutoCAD I/O', enables running scripts on native CAD design files such as `DWG`,
cf. [Albert's tutorials](https://github.com/szilvaa/acadio-tutorials).
- October 6
– [Introduction to BIM360](http://adndevblog.typepad.com/cloud_and_mobile/2016/10/introduction-to-bim-360.html) and
the [Forge DevCon recording on BIM 360](https://www.youtube.com/watch?v=cOQEyI-EMAQ)
– [BIM360](https://developer.autodesk.com/en/docs/bim360/v1/) enables apps to integrate with BIM360 to extend its capabilities in the construction ecosystem.
- October 11
– [Introduction to Fusion 360 Client API](http://adndevblog.typepad.com/manufacturing/2016/10/introduction-to-fusion-360-api.html)
– [Fusion 360 Client API](http://help.autodesk.com/view/NINVFUS/ENU/?guid=GUID-A92A4B10-3781-4925-94C6-47DA85A4F65A) implements an integrated CAD, CAM, and CAE tool for product development, built for the new ways products are designed and made;
resources:
[forum](http://forums.autodesk.com/t5/api-and-scripts/bd-p/22),
[samples](https://autodeskfusion360.github.io),
[learning material](http://fusion360.autodesk.com/learning/learning.html?guid=GUID-A18E7686-1C84-4690-95EE-E2076A1BD84E&_ga=1.102015364.466522287.1440788713)
For code samples on any of these, please refer to the Forge Platform samples on GitHub
at [Developer-Autodesk](https://github.com/Developer-Autodesk)
and [Autodesk-Forge](https://github.com/Autodesk-Forge),
optionally adding a filter, e.g., like this for `Design.automation`: [...Developer-Autodesk?query=Design.automation](https://github.com/Developer-Autodesk?query=Design.automation).
The Forge Webinar series continues during the remainder of
the [Autodesk App Store Forge and Fusion 360 Hackathon](http://autodeskforge.devpost.com) until the end of October:
- October 27 @ 8.00 AM PST – Submitting a web service app to Autodesk App store.
Please feel free to contact us at [forgehackathon@autodesk.com](mailto:forgehackathon@autodesk.com) at
any time with any questions you may have on Autodesk Forge or the Autodesk Fusion 360 API.
We will be happy to work with you to resolve your issues 1-on-1.
Please note that the Q&A session 2 originally scheduled for October 20th 8.00 AM PST has been cancelled, since we will instead be working directly with developers who have issues or questions.
Quick access links:
- For API keys, go to [developer.autodesk.com](https://developer.autodesk.com)
- For code samples, go to [github.com/Developer-Autodesk](https://github.com/Developer-Autodesk)
![Forge – build the future of making things together](img/forge_accelerator.png)