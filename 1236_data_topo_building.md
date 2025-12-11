---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.576307'
original_url: https://thebuildingcoder.typepad.com/blog/1236_data_topo_building.html
post_number: '1236'
reading_time_minutes: 11
series: general
slug: data_topo_building
source_file: 1236_data_topo_building.htm
tags:
- elements
- family
- filtering
- geometry
- levels
- parameters
- revit-api
- transactions
- views
- walls
- windows
title: Creating Topography Contours and Building Masses
word_count: 2286
---

### Creating Topography Contours and Building Masses

After diving deep into both Revit
[MEP](http://thebuildingcoder.typepad.com/blog/2014/10/newcrossfitting-connection-order.html) and
[Structure](http://thebuildingcoder.typepad.com/blog/2014/11/concrete-setout-points-for-revit-structure-2015.html)
API issues in the past few days, let's round it of with a GIS related topic:

I had an interesting discussion back in the spring of this year with
[Mohammad Rahmani Asl](https://sites.google.com/site/bimsimgroup/people/students/mohammad-rahmani-asl),
[Saied Zarrinmehr](https://sites.google.com/site/bimsimgroup/people/students/saied-zarrinmehr) and
[Chengde Wu](https://sites.google.com/site/bimsimgroup/people/students/chengde-wu)
of the
[Texas A&M University](https://www.tamu.edu) in the Revit API discussion forum on the topic of
[creating a height dimension for a form](http://forums.autodesk.com/t5/revit-api/creating-height-dimension-for-form/m-p/4886750).

I finally grabbed some time last weekend to document that discussion thread and publish their interesting application:
[DataToBim](https://github.com/jeremytammik/DataToBimCountoursAndBuildings),
a Revit add-in that creates topography, building outlines and solid conceptual mass extrusions from text file input data:

![Contours and masses](img/datatobim_contour_and_mass.png)

Here is the
[five and a half minute video](http://vimeo.com/103487058) describing
the final completed project:

The initial question from Mohammad was on creating the dimensioning required to extrude the building shape masses.

Later, Saied added background information and motivation on the application itself:

**Question:** I am creating an application that needs to create dimension for height of a form.
I read The Building Coder and the forum posts and used the sample in SDK folder (Window Wizard).
I was able to manage everything well but the following error:

```
  The direction of dimension is invalid.
```

I tried a lot to solve this issue, but I could not find the solution.
Two other people had the same problem in the blog but there is no answer for their questions.
I would appreciate if you could help me.

Here is [saieds\_program.zip](zip/datatobim_saieds_program.zip) containing the complete source code and Visual Studio solution together plus some sample data text files.

This application imports data from GIS to Revit.
When you run it from the add-in manager it creates the topography first and then adds the masses (using extrusion command) on the top of topography:

![Mass extrusion](img/datatobim_img_masses.png)

Please follow these steps:

1. Copy the TextForPartialMap folder to your “C:/” drive or copy it to any folder and change paths in the MainClass.Cs for Buildings Road and Contours.
2. Open and empty Revit Architecture Project and go to Level 1 View
3. Run the application from add in manager
4. The program creates the topography using the data in the text files included in the project folder (It takes a few seconds)
5. It will ask for a folder to save the mass files (select a folder and click OK)
6. Here comes the error:

![Error message](img/datatobim_img_error.jpeg)

The code attempts to add height dimension to the mass and label it with the height parameter (please see the DataToBuilding.cs then CreateFamilyFile Function when we try to create the dimension).

The error that we receive is highlighted with a few stars in comment in the code.

**Answer:** I had some trouble rebuilding your project.

I first tried to use the current version of the clipper library, clipper\_ver6.1.3a.zip.

That no longer defines the AddPolygons method, but AddPaths instead.

I then reverted back to a version I had used previously, clipper\_ver5.1.6.zip.

That does not yet define the StrictlySimple property.

After commenting out that in your code I was able to compile.

I tried running it, and encountered the error System.ArgumentException: "Could not find the path!"

That was due to your implementation of the familyTemplateAddress method, which only provides support for the imperial library.

I only have the metric library installed, so I added support for that.

I was able to reproduce the error that you run into, saying 'The direction of dimension is invalid. A transaction or sub-transaction was opened but not closed in document 'Family2'. All uncommitted changes to this document made by External Command will be discarded.'

Looking at the name of the view, I see 'RayTracer View', created by the CreateIsometric method your method createView3d.

Try adding a dimension to that view in the user interface and see what happens :-)

As far as I know, adding dimensioning requires a 2D view, and the dimensioning must lie in the view plane.

I would therefore suggest that you search for or create a suitable such view, presumably an elevation view, and use that to create the dimensioning.

Some samples are available on the net, e.g. by searching for [Revit API NewDimension](http://lmgtfy.com/?q=revit+api+NewDimension).

**Response** I appreciate your attention and the time that you spent on the issue that Mohammad reported.
Although I have never contacted you but I have read every single post of your blog.
I owe you a lot.
Let me know when you come over College Station, Texas to take you out to a restaurant :-)

Since you have spent quite a bit of time reading and trying to understand our code you might be curious to have an overview of the application. I am the main developer of an application in Revit that reads ArcGIS shape files and automatically creates a BIM model that contains GIS tabular information in the form of Revit parameters. Developing this application is only possible in Revit 2014 in which the API library includes constructors for SiteSubRegion and BuildingPadType.
This application is part of a research project in which the user interface also conducts queries in Revit to search, find and visualize building masses based on their parameter values. The UI also automates the creation of facades on the faces of a mass. The project is at its final steps.

My colleague, Mohammad, was working to connect the parameters to control the geometry of the mass. Since Mohammad and I were working on the same class "DataToBuilding.cs" we made some mistakes. The "ray tracer" view was created on the main project document to find the elevation of the building (which is not usually available in GIS). In this view rays will be shot vertically from the corners of the building footprint; the closet distances will be measured, and the average distances will be calculated to find the elevation of the building on the topography. Then we will create a new mass family and locate it on the topography. Here is where we used the address of the template family using imperial folder. We will create the new "Conceptual Mass" families in Revit API and that is where the issues are raised.

It is easy to create a new family document and create new extrusions using Revit API library. The corners of building footprints that are retrieved from GIS data are set according to the North American datum and the numbers might be big double values. As a result when you create a new extrusion in a mass family that is generated with API, the extrusion is very likely be placed far from the default visible range of the default view of a family. You can nonetheless load these families in the project and see their preview on the project browser when you want to place an instance of these families! Creating new dimensions also has the same view issue. A family document that is created by API does not have active view and manipulating the view properties to expand the range of visibility seems to be impossible maybe because there is not any view at all. I could solve the problem of family visibility by translating the building footprint to the canter of the family datum, creating the extrusion, saving the family, loading the family into the project file and finally translating the family back in its original location. This strategy worked to solve the problem of viewing the geometry of the family; however, it cannot be applied to take care of the dimension issue. Like many other geometric elements, dimensions in Revit are also view-dependent.

I can summarize our problem in this way, "how could we create a 2D elevation view in a family that was generated by Revit API?"

Thanks again for your attention. I would be happy to provide more detailed explanation of the application for your blog when the development is completed.

PS: In the previous model you can activate the visibility of masses to see the buildings.
Our results will look like this image showing part of the Texas A&M main campus:

![Texas A&M campus](img/datatobim_texas_a_m_campus.png)

The clipper library was used to perform polygon Boolean operations.
We also worked with a couple of other external libraries but when simplifying the application to send it to you we forgot to remove the clipper library.

Later:
To address the problem at hand, I now found The Building Coder discussion on [section view creation](http://thebuildingcoder.typepad.com/blog/2011/07/section-view-creation.html).

The Revit 2014 API doc.Create.NewViewSection method is now replaced by a static member of ViewSection class.
I guess viewSection is what we need to use.
Is that true?

Later still:
The problem is solved.
A new family document has a list of views that can be retrieved using a FilteredElementCollector and filtered to find views of type elevation, ViewType.Elevation.
We even did not need to generate any new views.

**Answer:** I am very glad to hear that you solved it.

The approach searching for an existing elevation view and adding the dimensioning to that sounds perfect, as long as you do not care specifically which view it is added to.

Yes, you would use the static ViewSection.CreateSection method nowadays instead of NewViewSection.
Here are some samples of using that:

- [Change Section View Type and Hide Cut Line](http://thebuildingcoder.typepad.com/blog/2012/05/change-section-view-type-and-hide-cut-line.html)
- [Create Section View Parallel to Wall](http://thebuildingcoder.typepad.com/blog/2012/06/create-section-view-parallel-to-wall.html)
- [Set View Section Box to Match Scope Box](http://thebuildingcoder.typepad.com/blog/2012/08/set-view-section-box-to-match-scope-box.html)
- [Language Independent Section View Type Id Retrieval](http://thebuildingcoder.typepad.com/blog/2013/08/language-independent-view-type-id-retrieval.html)

I later put together dedicated topic lists on
[creating dimensioning](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.45) and
[creating and setting up section views](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.38).

**Response:** Here is the updated and somewhat polished Visual Studio solution.

Unfortunately, I cannot release more from the on-going research that is not published yet.

I added the text files (data from GIS) to the resources of the VS project so that people do not get into the trouble of setting the folder names.

I also would like to add something about the method we used to create the TopographicSurface.
The static method to create an instance of TopographicSurface class asks for a list of XYZ points to create a mesh representing the topography in Revit. Unlike Revit, ArcGIS represents topography with contour curves. The contours in GIS can intersect with each other or be drawn on the top of each other, meaning that we may get points with identical X and Y values. The create method in TopographicSurface class will throw and exception when the list of points includes points with identical X and Y values. I followed two solutions to remove identical points: the geometric solution and the non-geometric solution.

In the geometric solution I recursively created rectangular subareas and in each subareas checked each pair of points to check if they have identical x and y values. This solution took days to implement and took hours to work on small topographic surface.
I decided to use the non-geometric solution taking advantage of the internal structure of a dictionary.
In my dictionary each key-value pair includes a key that is uniquely defined based on the X and Y of each XYZ point and the point itself as the value. When adding new key-values to the dictionary, if the keys are the same, meaning that the points are on the top of each other, the dictionary will throw an exception. To create an appropriate key I converted the X and Y values of each point to strings and concatenated the string values to create one unique string.
This method worked elegantly – fast and accurate.

**Answer:** I compiled and tested the original code on Revit 2014, publishing it in the
[DataToBimCountoursAndBuildings GitHub repository](https://github.com/jeremytammik/DataToBimCountoursAndBuildings) as
[release 2014.0.0.0](https://github.com/jeremytammik/DataToBimCountoursAndBuildings/releases/tag/2014.0.0.0).

I migrated it to Revit 2015 and created
[release 2015.0.0.0](https://github.com/jeremytammik/DataToBimCountoursAndBuildings/releases/tag/2015.0.0.0) as
a snapshot of that.

Finally, I cleaned it up a bit further, mainly by encapsulating all transactions in 'using' statements, because
[using using automagically disposes and rolls back](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html),
to create the current
[release 2015.0.0.1](https://github.com/jeremytammik/DataToBimCountoursAndBuildings/releases/tag/2015.0.0.1).

Don't forget to turn on visibility for Mass and Topography in the View > Visibility/Graphics overrides when testing.

Many thanks to Saied, Mohammad and Chengde for all their work and research on this!