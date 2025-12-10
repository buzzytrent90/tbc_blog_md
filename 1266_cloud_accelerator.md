---
post_number: "1266"
title: "Cloud Accelerator and More Revit Stuff"
slug: "cloud_accelerator"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'references', 'revit-api', 'transactions', 'views', 'windows']
source_file: "1266_cloud_accelerator.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1266_cloud_accelerator.html"
---

### Cloud Accelerator and More Revit Stuff

Last opportunity to make one more New Year's Resolution!

Submit a proposal for the cloud accelerator:

- [Cloud Accelerator](#1)
- [Revit API Discussion Forum Activity](#2)
- [C# Samples and Getting Started with the Revit API](#3)
- [2D Coordinates of Curve Endpoints in View3d](#4)
- [Edit IFC Import to Write Extensible Storage](#5)

#### Cloud Accelerator

Talking about opportunities to dive deep and fast into cool web oriented technology, check out the Autodesk
[Cloud Accelerator](http://www.autodeskcloudaccelerator.com),
a two-week workshop in March 2015 in our San Francisco office for up to 14 creative software developers.

We will be holding our first Cloud Accelerator program at the Autodesk San Francisco office in March this year. The Accelerator will allow you to spend quality time working on your project making use of Autodesk Cloud APIs while benefitting from help and advice from Autodesk API experts.
It’s an ideal opportunity to get away from the daily grind and work on something new.

To apply for this program, all you have to do is send us a 1000 word proposal outlining your proposed application. Due to popular demand, we extended the application deadline by a week to Saturday January 17th, updated from
[the initial announcement](http://thebuildingcoder.typepad.com/blog/2014/11/cloud-accelerator-vdc-and-transaction-groups.html#2).
That said, if you submit your proposal by the original deadline of January 10th, we will have time to provide you with feedback to increase the chance of your proposal being accepted.

If this sounds interesting, then please visit [autodeskcloudaccelerator.com](http://www.autodeskcloudaccelerator.com) for more information, or email or call us with your questions. We hope to see you in March.

For two weeks, March 9 to March 20, 2015, participants will work intensively on their chosen project with the help, support, and training of the Autodesk Cloud engineering teams. This includes daily classes by industry experts on important aspects of cloud development and one-on-one help from these experts in real time. At the end of the Accelerator, you will also have the opportunity to present your prototype to senior Autodesk executives.

![Autodesk Cloud Accelerator](img/cloud_accelerator.jpg)

Don't miss this opportunity to spend two weeks of quality time in sunny San Francisco working on your own exciting cloud project while benefitting from a wealth of expert help and advice.

#### Lots of Revit API Discussion Forum Activity

Back to the Revit API, I was very busy on the Revit API forum again yesterday:

- [C# samples](http://forums.autodesk.com/t5/revit-api/c-samples/td-p/5460197)
- [My first attempt at C#](http://forums.autodesk.com/t5/revit-api/my-first-attempt-at-c/td-p/5460202)
- [Automate adding parameters](http://forums.autodesk.com/t5/revit-api/automate-adding-parameters/m-p/5459266)
- [Macro user rights](http://forums.autodesk.com/t5/revit-api/macro-user-rights/m-p/5460913)
- [Override command within dialogue](http://forums.autodesk.com/t5/revit-api/override-command-within-dialogue/m-p/5459344)
- [Insert raster image](http://forums.autodesk.com/t5/revit-api/insert-raster-image/m-p/5459996)
- [2D coordinates of curve endpoints in View3d](http://forums.autodesk.com/t5/revit-api/2d-coordinates-of-curve-endpoints-in-view3d/m-p/5425365)
- [Edit IFC import to write extensible storage](http://forums.autodesk.com/t5/revit-api/edit-ifc-import-to-write-in-extensible-storage/m-p/5461577)

Several of them I find worthwhile repeating here:

#### C# Samples and Getting Started with the Revit API

**Question:** For the last couple of months I have been trying to learn C#.
It has not been easy but I’m slowly getting the hang of it.
Now I’m working on trying to migrate over to writing C# for Revit.
I have found a couple different samples but I’m looking for more.
Can anyone point me to a site that has lots of examples and maybe even more training videos, tutorials and/or workbooks?
Thanks.

**Answer:** Please start with the
[Revit API getting started material](http://thebuildingcoder.typepad.com/blog/about-the-author.html#2).

Work through the My First Revit Plugin and DevTV tutorials first.

They provide exactly what you need fast and efficiently.

Kwhite also recommends looking at the [Udemy](https://www.udemy.com) site.
Harry Mattison's class 'Learn to program the Revit API by Boost Your BIM' and his [blog site](https://boostyourbim.wordpress.com/) are very helpful.
There are a lot of reference sites out there.
The more you dive into the Revit API and perform searches using Google or any other preferred search engine, you will discover the multitude of resources available.

Also, without really needed to write this out, [The Building Coder](http://thebuildingcoder.typepad.com) is the most often the first source of my search for additional information.

I also have found the help wiki to be very informative, e.g. here, the
[English Revit 2015 wiki help](http://help.autodesk.com/view/RVT/2015/ENU).
At this link you will see a Developers area in the navigation tree.
Give that a peek as well.

Good luck with your C# endeavour.
It can be frustrating in the beginning, but very fulfilling once you get some momentum going.

Have fun!

#### 2D Coordinates of Curve Endpoints in View3d

I always love geometrical questions, even simple ones... this one maybe is not so trivial after all?

**Question:** I want to obtain 2D coordinates at the viewport of curve endpoints within a View3D.

Therefore, I filter the visible elements with the FilteredElementCollector. Is there a way to directly obtain the 2D coordinates on the screen without doing perspective transformation of the element to the camera coordinates and project it to the image plane?

**Answer:** Yes, there is.

You can use the
[UIView class](http://thebuildingcoder.typepad.com/blog/2012/06/uiview-and-windows-device-coordinates.html) and
its two methods GetZoomCorners and GetWindowRectangle.

The former returns Revit XYZ coordinates, the latter a Rectangle on the screen.

I show how to make use of this in my
[custom tooltip](http://thebuildingcoder.typepad.com/blog/2012/10/uiview-windows-coordinates-referenceintersector-and-my-own-tooltip.html).

#### Edit IFC Import to Write Extensible Storage

**Question:** I successfully modified the IFC export to add my own extensible storage data.

Now I would like to modify the IFC import to read it back in.

How can this be achieved, please?

In detail:

I wrote some data into Revit Extensible Storage.
For interchanging Revit models with other Systems we use the IFC format.
To write the Extensible Storage Data to an IFC File I edited the [SourceForge-IFC-Project](http://sourceforge.net/projects/ifcexporter) as I needed and my data is written as PropertySets into the IFC File.
Everything fine.
Now the question, how to get the data back into Extensible Storage when Importing the IFC File.
Do you have any idea?
I didn't find where to edit the IFC Import, for the SourceForge Project just calls the method Autodesk.Revit.DB.IFC.ImporterIFC.ProcessIFCProject provided by the class Revit.IFC.Import.Data.IFCImportFile, and it does all the work.
Can you help me please and tell, where I can customize the IFC-Import the way I want to?

Thank you!

**Answer:** Congratulations on adding your own data to the IFC export.

The Open Source only really supports Export and Link IFC, not Import IFC.

That said, you can still modify the open source to work in this case as well.

In particular, after the call to ProcessIFCProject() (which basically does the entire import IFC), you can go through the document, search through the created elements and get their IFC GUID. When you find the matching IFC GUID, you can add whatever data you want.

We are thinking about supporting Open IFC more fully in open source as well in the future...