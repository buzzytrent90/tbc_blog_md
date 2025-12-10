---
post_number: "1219"
title: "ADVA Webinar, Free Student Software and AU"
slug: "adva_free_au"
author: "Jeremy Tammik"
tags: ['csharp', 'python', 'revit-api', 'rooms', 'transactions', 'views']
source_file: "1219_adva_free_au.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1219_adva_free_au.html"
---

### ADVA Webinar, Free Student Software and AU

Here are pointers to some relevant non-API news items of interest:

- [Free webinar introducing the new Autodesk View and Data API](#2)
- [Free Autodesk software for all students everywhere](#3)
- [AU class enrolment](#4)

#### Free Webinar – Introducing the new Autodesk View and Data API

We already held a webinar on the new Autodesk View and Data API in September.

Here are the old
[description and full details](http://thebuildingcoder.typepad.com/blog/2014/09/autodesk-view-and-data-api-webinar.html#3) of that, and, for the first time, we proudly present the
[detailed notes of the September presentations](zip/2014-09-18_adva_webcast_notes.pdf) by Stephen Preston and Cyrille Fauvel.

In case you missed it, another chance is coming up again soon: the next free webinar
[Introducing the new Autodesk View and Data API](http://adndevblog.typepad.com/cloud_and_mobile/2014/10/free-webinar-introducing-the-new-autodesk-view-and-data-api-.html) will
be held on Thursday October 23rd, 2014.

#### Free Autodesk Software for All Students Everywhere

Autodesk now provides free software for students, teachers, and schools around the world.

Inspiring creativity and a love for science, engineering, and math in young students is more important today than ever. So I'm very happy to announce that Autodesk has opened up FREE access to its desktop software to ALL students, teachers, and schools everywhere in the world.

In the past, access was free in some countries, but not others. Today there are no restrictions. However, all too often students and teachers are unaware that they can have their own copy and use our software in their classroom for free. The simplest way to help a student or teacher is to direct them to
[students.autodesk.com](http://www.autodesk.com/education/home) where
they can download as much as they want and find curriculum and learning materials.

#### AU Class Enrolment

I mentioned the Revit API expert panel
[SD5156 – Open House on the Factory Floor](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=5156) taking place at Autodesk University.

![Autodesk University 2014](img/AU14-Speaker-275x250.png)

A few of my fellow ADN colleagues' class enrolments are currently below the attendee threshold, however:

- [**SD6310**](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=6310) – Bring on the Cloud and Mobilize Your Media & Entertainment Workflows Through Programming – Kevin Vandecar
- [**SD6432**](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=6432) – Are You Having a Pythonic 3ds Max Experience? No? Come Join the Revolution! – Kevin Vandecar
- [**SD5752**](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=5752) – OAuth 1.0 Versus OAuth 2.0 and Use Case – Cyrille Fauvel
- [**SD6230**](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=6230) – Add 3D Photogrammetry to Your Desktop and Mobile Apps using ReCap Photo API – Philippe Leefsma
- [**SD5931**](https://events.au.autodesk.com/connect/sessionDetail.ww?SESSION_ID=5931) – Creating Commercial Applications for Mobile Devices Using AutoCAD OEM – David Grieve

If these classes are of interest to you, please register for them to ensure they really take place.

You can check the
[AU 2014 class catalogue](https://events.au.autodesk.com/connect/dashboard.ww) for
the complete list of classes.

---

# AEC DevBlog

### Adding TrimPlanes to Structural Members using ACA .NET

By
[Jeremy](http://adndevblog.typepad.com/cloud_and_mobile/jeremy-tammik.html)
[Tammik](http://thebuildingcoder.typepad.com/blog/about-the-author.html).

**Question:** How can I add TrimPlanes to structural members using ACA .NET API?

I tried the
[code provided in 2012](http://adndevblog.typepad.com/aec/2012/12/adding-trimplanes-to-structural-members-using-aca-net-api.html) with little success.

When I try to write to the TrimPlanes property, e.g. like this, it returns a 'ReadOnly'" error:

```csharp
member.TrimPlanes = trimPlanes
```

Any ideas how to fix this?

**Answer:** Basically, the following sample code skeleton should work:

```csharp
  trimPlanes= owner.TrimPlanes;

  plane1 = new TrimPlane();
  ...
  trimPlanes.Add(plane1);
```

Here is a complete command implementation sample:

```vbnet
    <CommandMethod("testmemberTrim")> \_
    Public Sub testmemberTrim()
      Dim db As Database = HostApplicationServices.WorkingDatabase
      Dim tm As Autodesk.AutoCAD.DatabaseServices.TransactionManager = db.TransactionManager
      Dim trans As Transaction = tm.StartTransaction()
      Dim ed As Editor = Application.DocumentManager.MdiActiveDocument.Editor
      Try
        Dim member As Member = New Member()
        member.MemberType = MemberType.Column
        member.SetDatabaseDefaults(db)
        member.SetToStandard(db)
        ' Set the start and end point of Member in WCS
        member.Set(New Point3d(0.0, 0.0, 0.0), New Point3d(5000.0, 0.0, 0.0))
        ' create a trim plane at the start
        Dim ptOrigin As Point3d = New Point3d(1.0, 1.0, 1.0)
        Dim vec As Vector3d = New Vector3d(1, 0, 0)
        Dim tp1 As TrimPlane = New TrimPlane()
        tp1.SubSetDatabaseDefaults(db)
        tp1.SetToStandard(db)
        tp1.End = TrimPlaneFrom.Start
        tp1.Plane = New Plane(ptOrigin, vec.GetNormal())
        member.TrimPlanes.Add(tp1)
        ' create another trim plane at the end
        Dim tp2 As TrimPlane = New TrimPlane()
        tp2.SubSetDatabaseDefaults(db)
        tp2.SetToStandard(db)
        tp2.End = TrimPlaneFrom.End
        tp2.Plane = New Plane(ptOrigin, vec.GetNormal())
        member.TrimPlanes.Add(tp2)
        Dim blkTbl As BlockTable = trans.GetObject(db.BlockTableId, OpenMode.ForRead)
        Dim ms As BlockTableRecord = trans.GetObject(blkTbl(BlockTableRecord.ModelSpace), OpenMode.ForWrite)
        ms.AppendEntity(member)
        trans.AddNewlyCreatedDBObject(member, True)
        trans.Commit()
      Catch
        MsgBox("\nMember creation failed")
        trans.Abort()
      Finally
        MsgBox("\nMember created!")
        trans.Dispose()
      End Try
    End Sub
```
![Column trim planes](img/column_trim_planes.png)