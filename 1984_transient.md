---
post_number: "1984"
title: "Transient"
slug: "transient"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'references', 'revit-api', 'selection', 'sheets', 'transactions', 'views']
source_file: "1984_transient.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1984_transient.html"
---

### Lookup Ideas, Jigs and ACC Docs Access
Today, we look at a request for new ideas for enhancing RevitLookup, implementing a pickpoint rubber band and opening BIMs on ACC Docs:
- [Request for RevitLookup ideas](#2)
- [Transient elements for jig](#3)
- [Transient `DirectShape` jig](#3.1)
- [Opening a model in ACC Docs](#4)
- [Stop using JPEG](#5)
- [Stop using voice id](#6)
#### Request for RevitLookup Ideas
Do you have any ideas
for [RevitLookup](https://github.com/jeremytammik/RevitLookup) enhancements?
A lot of exciting functionality has already been worked on in
the [dev](https://github.com/jeremytammik/RevitLookup/tree/dev)
and [dev_winui](https://github.com/jeremytammik/RevitLookup/tree/dev_winui) branches.
We expect to see that coming out quite soon.
Meanwhile, Roman [Nice3point](https://github.com/Nice3point) opened a discussion for collecting
[RevitLookup Ideas](https://github.com/jeremytammik/RevitLookup/discussions/146).
Your contributions there are welcome.
Thank you!
#### Transient Elements for Jig
A couple of ideas on creating transient elements graphics similar to the AutoCAD jig functionality using
the `IDirectContext3DServer` functionality or the temporary InCanvas graphics API were recapitulated in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [drawing line visible on screen](https://forums.autodesk.com/t5/revit-api-forum/draw-line-visible-on-screen/m-p/11778165):

* [Onbox, DirectContext Jig and No CDN](https://thebuildingcoder.typepad.com/blog/2020/10/onbox-directcontext-jig-and-no-cdn.html)
* [Transient Graphics, Humane AI, BI and Lockdown](https://thebuildingcoder.typepad.com/blog/2021/01/transient-graphics-humane-ai-basic-income-and-lockdown.html)
* [Flip, Mirror, Transform and Transient Graphics](https://thebuildingcoder.typepad.com/blog/2021/05/flip-mirror-transform-and-transient-graphics.html)

![Pick point rubber band](img/pick_point_rubber_band.png "Pick point rubber band")
Lorenzo Virone shared a [different approach](https://forums.autodesk.com/t5/revit-api-forum/draw-line-visible-on-screen/m-p/11778165#M69522), creating and deleting database-resident Revit elements on the fly in a loop:
I faced a similar UI problem to create a rubber band between two points.
I used two functions, `Line.CreateBound` and `NewDetailCurve`, inside a loop to create a line at the cursor position, refresh, and delete the line every 0.1 seconds, until the user chooses the second point.
A little tricky, but it works fine for me, and Revit seems to execute these 2 functions very fast.
This trick will technically work with anything:
create new elements on each mouse movement, refresh, delete the created elements and replace them with new ones.
You can use either model or detail elements.
It's easy to implement, because you just need to call the two methods, e.g., like this:

28. bool done = false;
29. List temp = new List();
31. while (!done)
32. {
33. doc.Delete(temp);
35. // Create temp elements
36. // Save their IDs in `temp`
37. // Set `done` to `true` when finished
39. doc.regenerate();
40. uidoc.RefreshActiveView();
41. Thread.Sleep(500); // milliseconds
42. }
44. // Your final elements are in `temp`

Many thanks to Lorenzo for sharing this nice solution.
#### Transient DirectShape Jig
Chuong Ho adds: This technique can also be used with a `DirectShape` element:

1. using Autodesk.Revit.DB;
2. using Autodesk.Revit.UI.Selection;
3. using System.Collections.Generic;
4. using Line = Autodesk.Revit.DB.Line;
5. using Point = Autodesk.Revit.DB.Point;
7. var Doc = commandData.Application.ActiveUIDocument.Document;
9. using TransactionGroup trang = new TransactionGroup(Doc, "test");
10. trang.Start();
11. XYZ a = UIDoc.Selection.PickPoint(ObjectSnapTypes.None);
12. SetPoint(a);
13. XYZ b = UIDoc.Selection.PickPoint(ObjectSnapTypes.None);
14. SetPoint(b);
15. SetLine(a, b);
16. XYZ p1 = UIDoc.Selection.PickPoint(ObjectSnapTypes.None);
17. SetPoint(p1);
18. XYZ p2 = UIDoc.Selection.PickPoint(ObjectSnapTypes.None);
19. SetPoint(p2);
20. bool isSamSide = IsSamSide(p1, p2, a, b);
21. MessageBox.Show(isSamSide.ToString());
22. trang.Assimilate();
24. // visualize a point
25. void SetPoint(XYZ xyz)
26. {
27. using (Transaction tran = new Transaction(Doc, "Add point"))
28. {
29. tran.Start();
30. Point point1 = Point.Create(xyz);
31. DirectShape ds =
32. DirectShape.CreateElement(Doc, new ElementId(BuiltInCategory.OST_GenericModel));
33. ds.SetShape(new List() { point1 });
34. tran.Commit();
35. }
36. }
38. // visualize a line
39. void SetLine(XYZ x1, XYZ x2)
40. {
41. using (Transaction tran = new Transaction(Doc, "Add line"))
42. {
43. tran.Start();
44. Line line = Line.CreateBound(x1, x2);
45. DirectShape ds = DirectShape.CreateElement(
46. Doc, new ElementId(BuiltInCategory.OST_GenericModel));
47. ds.SetShape(new List() { line });
48. tran.Commit();
49. }
50. }

![Pick point rubber band](img/pick_point_rubber_band_jig_directshape.gif "Pick point rubber band")
Many thanks to Chuong Ho for this addition!
#### Opening a Model in ACC Docs
We started out
discussing [opening a cloud model with Revit API](https://forums.autodesk.com/t5/revit-api-forum/opening-a-cloud-model-with-revit-api/m-p/11767222) in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
but then moved it over to StackOverflow, the better place for such a cloud-related topic, where my colleague Eason Kang explains
how to [open files located in ACC Docs](https://stackoverflow.com/questions/75530623/open-files-located-in-the-accdocs):
\*\*Question:\*\* My Visual Studio Revit API add-in open Revit files to export data in a batch.
I can add many files which are on the networks and the plugin automatically opens them all.
Is it possible to also open files that are in the ACC Docs cloud?
I know I can open AccDocs which were be already downloaded locally by searching for them in the collaboration cache folder, but how to open files which have not yet been downloaded?
\*\*Answer:\*\* Since you mention the collaboration cache folder, I assume you are using
the [Revit Cloud Worksharing model](https://knowledge.autodesk.com/support/bim-360/learn-explore/caas/CloudHelp/cloudhelp/ENU/About-BIM360/files/about-bim-360-design/About-BIM360-about-bim-360-design-about-revit-cloud-worksharing-html-html.html),
a.k.a. `C4R`, the model for Autodesk Collaboration for Revit.
If so, you can make use of
the [APS Data Management API](https://aps.autodesk.com/en/docs/data/v2/developers_guide/overview/) to
obtain the `projectGuid` and `modelGuid` in the model version tip like this:

```
{
   "type":"versions",
   "id":"urn:adsk.wipprod:fs.file:vf.abcd1234?version=1",
   "attributes":{
      "name":"fileName.rvt",
      "displayName":"fileName.rvt",
      ...
      "mimeType":"application/vnd.autodesk.r360",
      "storageSize":123456,
      "fileType":"rvt",
      "extension":{
         "type":"versions:autodesk.bim360:C4RModel",
         ....
         "data":{
            ...
            "projectGuid":"48da72af-3aa6-4b76-866b-c11bb3d53883",
            ....
            "modelGuid":"e666fa30-9808-42f4-a05b-8cb8da576fe9",
            ....
         }
      }
   },
   ....
}
```

With those in hand, you can open the C4R model using Revit API like this:

```
  // where is your BIM360/ACC account based, US or EU?

  var region = ModelPathUtils.CloudRegionUS;

  var projectGuid = new Guid("48da72af-3aa6-4b76-866b-c11bb3d53883");
  var modelGuid = new Guid("e666fa30-9808-42f4-a05b-8cb8da576fe9");

  // For Revit 2023 and newer:

  var modelPath = ModelPathUtils.ConvertCloudGUIDsToCloudPath(
    region, projectGuid, modelGuid );

  // For Revit 2019 - 2022:

  //var modelPath = ModelPathUtils.ConvertCloudGUIDsToCloudPath(
  //  projectGuid, modelGuid );

  var openOptions = new OpenOptions();

  app.OpenAndActivateDocument( modelPath, openOptions ); // on desktop

  // on Design Automation for Revit or
  // to not activate the model on Revit desktop:

  // app.OpenDocumentFile( modelPath, openOptions );
```

You can also make use
the [Visual Studio APS Data Management package on NuGet](https://www.nuget.org/packages/Autodesk.Forge) for this.
The [Hubs Browser tutorial](https://tutorials.autodesk.io/tutorials/hubs-browser/) demonstrates its use.
References:
- [Accessing BIM 360 design models on Revit](https://aps.autodesk.com/blog/accessing-bim-360-design-models-revit)
- [How to open a cloud model](https://thebuildingcoder.typepad.com/blog/2020/04/revit-2021-cloud-model-api.html#4.4)
- [Developer's guide online help on cloud models](https://help.autodesk.com/view/RVT/2023/ENU/?guid=Revit_API_Revit_API_Developers_Guide_Introduction_Application_and_Document_CloudFiles_html)
Many thanks to Eason for this comprehensive answer!
#### Stop Using JPEG
Moving away from Revit and its API to other interesting current news, Daniel Immke suggests
that [it’s the future – you can stop using JPEGs ](https://daniel.do/article/its-the-future-stop-using-jpegs) and
presents an overview and rationale for some compelling alternatives, e.g., AVIF and WebP.
#### Stop Using Voice Id
Joseph Cox describes [how he broke into a bank account with an AI-generated voice](https://www.vice.com/en/article/dy7axa/how-i-broke-into-a-bank-account-with-an-ai-generated-voice)
– some banks tout voice ID as a secure way to log into your account.
He proves it's possible to trick such systems with free or cheap AI-generated voices.