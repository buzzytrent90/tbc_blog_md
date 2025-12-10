---
post_number: "1276"
title: "Family Instance Area and Auto-Loading a Project File"
slug: "inst_area_autoload"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'revit-api', 'rooms', 'views']
source_file: "1276_inst_area_autoload.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1276_inst_area_autoload.html"
---

### Family Instance Area and Auto-Loading a Project File

I have been pretty busy answering queries on the Revit API discussion forum in the last few days.

Here are two that I quite like that came up today, on
[calculating areas occupied by furniture and equipment family instances](#2) and
[loading a project file automatically on Revit start-up](#3).

#### Calculating Areas Occupied by Furniture and Equipment Family Instances

This query was raised by Vyom Dixit on
[how to calculate model group elements area](http://forums.autodesk.com/t5/revit-api/how-to-calculate-model-group-elements-area-using-revit-api/td-p/5492863):

**Question:** I am new to the Revit API. I need to know about following things:

1. How to calculate model group elements area using Revit API?
2. Is this possible using ExtrusionAnalyzer? If yes, then how we can achieve this?

As far as I know, ExtrusionAnalyzer needs a solid type object to draw the shadow, correct me if I am wrong.

From a model group I have extracted the elements that belong to the model group.
Now the problem is that extracted elements can have different types not necessarily solid type.

Let me know if anyone has any idea about this.

Thanks.

**Answer:** It really all depends on what type of elements you are talking about.

**Response:** I have model group that can have some equipment like Freezer, WorkTable or any Speciality Equipment, furniture etc.
I want to calculate the total area occupied by this equipment belonging to the model group.
Can you suggest any way to achieve this?

**Answer:** Once you put it like that, I understand what you need.

I have in fact solved that exact issue, for my
[room editor app](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.21), to determine
[boundary loops for equipment and furniture family instances](http://thebuildingcoder.typepad.com/blog/2013/04/extrusion-analyser-and-plan-view-boundaries.html).

In that case, I did in fact use the ExtrusionAnalyzer class to determine the closed loops.

In my case, though, it would probably have been easier just using the 2D plan view of the family instances.

For areas in general, though, the extrusion analyser is probably the right way to go.

At least it is a very generic solution.

As you say, it does indeed require a solid to work on.

If you do not have any solids, you will need to determine a closed 2D polygon from the plan view family instance geometry.

You could determine such a closed 2D polygon either by calculating the
[convex 2D hull](http://thebuildingcoder.typepad.com/blog/2009/06/convex-hull-and-volume-computation.html) or using a
[2D polygon library](http://thebuildingcoder.typepad.com/blog/2013/09/boolean-operations-for-2d-polygons.html).

Once you have a closed polygon, calculating its
[2D area](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html) is pretty straightforward.

**Response:** Many thanks to your quick response.
I have reviewed your post as you mentioned in your last reply.
In my case, I don't think extrusion analyser approach would work for all the model group elements becuase I am getting an exception to union some solids that belong to family instance.
So, I want to use the approach you mentioned in your post, "using 2D plan view of the family instances".
Can you suggest this approach in steps not the code?

**Answer:** Please note that I ran into such exceptions when performing the Boolean unions as well and added exception handlers to take care of those cases anyway.

The extrusion handler is fed with 3D solids retrieved from the family instances in a 3D view.

If you retrieve their geometry via a 2D plan view instead, you will get other results and other geometry, presumably mainly consisting of curves.

- Element.Geometry Property
- Options Class
- Options.View Property
- The view used for geometry extraction.

Grab those curves and do what thou wilt.

#### Loading a Project File Automatically on Revit Start-up

This query was raised by Davix on
[how to load a RVT file automatically when starting Revit](http://forums.autodesk.com/t5/revit-api/how-to-load-a-rvt-file-automatically-when-start-the-revit/m-p/5494468):

**Question:** I wonder how can I load a Revit file automatically when starting the Revit system.

Since the UIControlledApplication is the only parameter of the OnStartup function of the IExternalApplication class, while the OpenAndActivateDocument is the function of the UIApplication but not of the UIControlledApplication.

How can I invoke the UIApplication.OpenAndActivateDocument method in the OnStartup function? Or is there some other method to address the problem?

**Answer:** The UIControlledApplication controlled application provided in the OnStartup method cannot open a document, because Revit is still in the process of starting up and is not ready to open and process project files yet.

In the OnStartup method, you can set up a system to open the project document later on, when Revit has finished loading.

Before getting to that, though, let me point out that the easiest way to start up Revit with a project file loaded is to use
[Process.Start](http://thebuildingcoder.typepad.com/blog/2010/03/using-processstart-to-open-a-project-or-family.html).

Once Revit is up and running, as you have pointed out yourself, you can also use the UIApplication.OpenAndActivateDocument method, as demonstrated in this discussion on
[closing the active document](http://thebuildingcoder.typepad.com/blog/2012/12/closing-the-active-document.html).

You can prepare a call to this method from within the OnStartup method by temporarily subscribing to the Idling event in the OnStartup method, then unsubscribing from Idling in the Idling event handler, opening the document and doing whatever else you want at that point.

That should give you a couple of useful options to choose from.

**Answer** by Arnošt Löbel: The Revit API now provides another solution for the problem besides using the Idling event. Three years ago, the Idling event may have been the only work-around, but there is a better solution now: a better way of opening a document on start-up is via the ApplicationInitialized event. Like with the Idling event, an application subscribes to it during the OnStartup call. Revit will raise this event a bit later, when the Revit application start-up has completed. Remember, just as with the Idling event, the OpenAndActivateDocument method can be used only as long there is no active document in Revit yet.

#### Updated ADN Revit API Samples GitHub Repository

I continued working my way through the various Revit API GitHub repositories, eliminating obsolete API usage and updating the sample code copyright years from 2014 to 2015 in readiness for more significant updates later this year.

Yesterday, I updated the
[AdnRevitApiLabsXtra repository](https://github.com/jeremytammik/AdnRevitApiLabsXtra),
including the
[standard ADN Revit API training labs](https://github.com/ADN-DevTech/RevitTrainingMaterial)
([zip](http://images.autodesk.com/adsk/files/Revit2015APITraining.zip))
provided by the
[Revit Developer Centre](http://www.autodesk.com/developrevit).

#### Avoid Retrieving a Parameter by Name

The last couple of fixes that I applied dealt with the
[ten warnings](zip/adn_migr_2015_a.txt) caused
by calls to the deprecated Element.Parameter property taking a parameter name as an argument.

This property is now converted to a method named GetParameters returning a list of parameters, since multiple parameters can have the same name.

Retrieving a parameter by name has always been bad practice, since it is language dependent and less efficient than using the other methods, if they apply, e.g. by specifying the parameter definition object, a built-in parameter enumeration value, or a shared parameter GUID.

The only parameters that really have to be retrieved by name are custom family parameters.

From now on, to retrieve a parameter by name, you either call the new GetParamters method returning a list and pick the one you need.

Alternatively, you can call the LookupParameter method that will return the first one encountered and pray that it really is the one you expect.

#### Jobs@Autodesk

I just received a message pointing out two exciting new European job opportunities at Autodesk:

1. Vertical Solution Sales – Simulation – Nordics/UK – REQ #15WD17251:

   We have a fantastic opportunity for someone to be involved in developing the simulation market with cloud based technology. With our heavy investment in technologies such as composites, a key theme in the simulation world, we are hoping to kick start and drive fast growth in an exciting market. Do you have knowledge in composite, analytics or simulation sales? Are you interested in being part of a team to spearhead this market with a new innovative approach?
2. Territory Sales Manager – AEC UK/Benelux – REQ #15WD17282:

   We have an opportunity for a Territory Sales Manager to head up a hugely successful team focusing on the AEC market. This role will be based in the UK and also cover the Benelux region. Do you have what it takes to maintain the momentum?

Please get in touch.