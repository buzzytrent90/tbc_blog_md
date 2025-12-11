---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:15.306556'
original_url: https://thebuildingcoder.typepad.com/blog/1101_family_inst_perf.html
post_number: '1101'
reading_time_minutes: 13
series: family
slug: family_inst_perf
source_file: 1101_family_inst_perf.htm
tags:
- doors
- elements
- family
- filtering
- geometry
- levels
- parameters
- python
- references
- revit-api
- rooms
- sheets
- transactions
- views
- walls
title: View and Family Implementation, Booleans and Performance
word_count: 2673
---

### View and Family Implementation, Booleans and Performance

Certain applications require large numbers of similar but different objects, e.g. for framing, scaffolding, steel detailing etc.

Every one of the past few years, Revit has been further enhanced to better still support these kinds of applications and handle larger numbers of families and instances gracefully and performantly. This work is still on-going.

Below, we address a bunch of questions related to creating a new wooden structural BIM application.

One aspect of such applications is the need to generate a great number of drawings to display all the wooden structural elements of a building in the required form:

- Floor plans, one or more for each level
- Roof plans, one or more for each roof
- Building sections
- Individual wall plans, at least one for each wall, could be 2-3 and even ten and more
- Individual beam plans, one for each different type of beam in the building

Here is a list of questions, based on the understanding that Revit represents the real BIM objects in the drawings, so they are always up-to-date with the model.

**1)** If I want to represent the real objects in the plans, I understand that I have to generate a view for every element I want to represent on the drawing and place the desired views on a sheet.

**1.1)** How do I generate the view of a single element (e.g. a rafter of the roof) so that only this beam is visible in this view? The SketchPlane of the view should have any orientation in space (could be parallel to a roof face). Is this possible?

**Answer:** Yes, this is absolutely possible. You can create a separate view for each element, on the fly or permanently. You can set up that view completely as you wish, e.g. for the desired orientation and cut planes. In that view, you can hide all other elements and generate a plan of the single object of interest. I presented examples of using this technique to
[export walls and floors to individual SAT files](http://thebuildingcoder.typepad.com/blog/2012/01/export-walls-and-floors-to-sat.html) and
[generate DXF files for CNC fabrication of wall parts](http://thebuildingcoder.typepad.com/blog/2013/12/driving-cnc-fabrication-and-shared-parameters.html).

**1.2)** I also want to add some annotations (dimensions, texts) in this view. Dimensions should be up-to-date with the measured object. I suppose that choosing the correct references automatically solves this problem. In the centre of the beam appears a symbol for the length direction, represented by three blue lines forming a Z shape. This annotation (DetailCurve?) should be kept automatically up-to-date with the geometry of the beam, e.g. be always in the centre of the beam and parallel with the axis of the beam. How can I solve this?

**Answer:** Yes, setting up the correct references keeps the dimensioning up to date.
If you wish to apply other automatic adjustments, you can achieve that using the
[dynamic model updater framework DMU](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.31),
as demonstrated by the DynamicModelUpdate associative section view SDK Sample.

**1.3)** How can I display several perspectives of the same element, e.g. front, rear, side and section?

**Answer:** Generate an individual view for every perspective and arrange the views on the sheet.

**1.4)** Does the system support such a big number of views? There could easily be several hundreds for the whole building.

**Answer:** Yes.

**1.5)** Is it possible to prevent the browser displaying this huge amount of views?

**Answer:** Yes, there are several ways to achieve this.
Please take a look at the interesting thread on
[implementing custom view browsers](http://forums.autodesk.com/t5/Revit-API/Creating-custom-Project-Browser-plugin-Views-apply-Grouping-and/td-p/4791957) in
the Revit API discussion forum, and the links mentioned there, e.g. to the
[Kiwi Codes Project Browser](http://youtu.be/JqRV6SLjJrM) demo.

**2)** I also imagine another solution to generate the drawing above in Revit could be to generate ModelCurves (or rather DetailCurves directly in the view?) out of the proper projections of the beam, arrange these curves to form the desired layout and add necessary annotations (something like one single view containing everything in the drawing above) and place this view on a sheet. Would such a solution be possible? The disadvantage is that in this situation not the real objects would be represented in the drawings, so I have to manage myself keeping the drawings up-to-date of with the model.

The system we are developing now uses the approach described in 2) but we would prefer to represent the real model in the drawings.

**Answer:** This is theoretically possible but extremely hard to accomplish, and you would basically be fighting the system instead of making proper use of it. I totally understand where you are coming from, though. In traditional CAD programming, you get used to having total control and implementing exactly what you need for yourself. In Revit, you should try to forget about all that, especially when
[porting an AutoCAD application](http://thebuildingcoder.typepad.com/blog/2012/10/porting-an-autocad-application.html).
You really need to understand the
[difference between BIM versus free geometry](http://thebuildingcoder.typepad.com/blog/2012/02/bim-versus-free-geometry-and-product-training.html)
before embarking upon a Revit application project.
Make sure you have proper in-depth product training and really understand the customer needs and optimal best practice workflows.

**3)** Generating dimensions for the beam in the drawing above, the references for the dimension are special points of the geometry generated by machining. What possibilities do I have to identify these special points? Could I attach some special information to them when they are created, so that I can find them by this information?

**Answer:** Interesting question. You can query the geometry of the element, and use geometrical properties to determine the required reference. You can also convert references to stable representations and back again using the ConvertToStableRepresentation and
ParseFromStableRepresentation methods on the Reference class.

**3b)** If a new machining appears on the beam, or if an existing machining is deleted, the dimension has to be changed. Can I add new references or delete references from an existing dimension or do I have to delete the dimension and create a new one with the same line as the old one?

**Answer:** It is probably simplest and maybe necessary to recreate the dimensioning from scratch.

**4)** Can I move the line of an existing dimension (in order to prevent it from overlapping with other elements)? Could I even change the direction of the line?

**Answer:** I believe so. It may be necessary to recreate the dimension from scratch, though.

**5)** (incomplete, skipped).

**6)** While debugging, do I have any chance to see the coordinates of the two endpoints of a bound line in the Quick Watch dialog in Visual Studio?

**Answer:** One easy way to see all the data you are interested in at a glance is to create separate variables for them.
Also, a very powerful way to debug and
[interact intimately with the model and API simultaneously](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html) is
to use the Revit Python or Ruby scripting shells.

**7)** While debugging, can I inspect the geometry (displaying coordinates of end points of edges) of a family instance in the Quick Watch dialog in Visual Studio? If this is not possible, is there any tool to help me do this? I tried with RevitLookup, but there I only see the local coordinates of the geometry, not the global ones, after the transform is applied.

**Answer:** Please look at the
[RevitLookup update on GitHub](http://thebuildingcoder.typepad.com/blog/2014/01/lots-of-clouds.html#6).
It includes extended functionality for snooping of geometry, FormatOptions and RevitLinkInstances.
I also developed some own tools in that direction, e.g. the
[GeoSnoop .NET boundary curve loop visualisation](http://thebuildingcoder.typepad.com/blog/2013/04/geosnoop-net-boundary-curve-loop-visualisation.html) tool
used for my room editor
[room and furniture loops](http://thebuildingcoder.typepad.com/blog/2013/04/room-and-furniture-loops-using-symbols.html) and
[structural framing cross sections](http://thebuildingcoder.typepad.com/blog/2013/12/security-framing-cross-section-analyser-and-rex.html).
You will probably have to develop some of you own as well to achieve exactly what you need.
I hope you implement the in a fashion generic enough to be useful and shareable with the entire Revit add-in developer community :-)

**8)** In the Revit SDK 2014 Samples, the transaction starts very early and includes all preparations for the object to be generated into the database. In other examples I saw that the transaction starts only before a database-resident object is about to be created, all preparations take place before the transaction starts. What is the best practice here?

**Answer:** I use the latter approach and would recommend that to you as well.

**9)** When I create a CurveArray or CurveArrArray using the Application.Create class, memory is allocated directly in the Revit application and not in the memory of add-in application; what are the reasons to use the Revit application memory and not allocate these collections directly using new?

**Answer:** "Ours is not to wonder why – ours is but to do or die." That is a design decision made by the Revit API team. Oh yes, it may also have to do with sharing objects and code between the external .NET memory space and the internal Revit universe. In general, we are trying to align the API more closely with the Revit internals, and also empower the API to enable parts of Revit to be implemented based on it, i.e. migrated out from the Revit kernel into the add-in space. This is an important step towards enabling more teams to work together more independently on different areas of the Revit product, similar to the kind of development separation and independence that ObjectARX enabled for AutoCAD.

**10)** It is possible to modify the geometry of a beam directly in the project model? Generally, our beams will contains a lot of machinery and almost all beams will have a different geometry. We have only been able to modify the geometry of a beam by modifying the geometry of the family defining it. In that case we should create for each particular beam a new family and there process all changes in the family geometry. There is any other way to do several changes in the geometry of a FamilyInstance object – JointGeometryUtils in not enough for us?

**Answer:** Currently, the only way to modify geometry in the fashion you describe in the project environment is the use of void cuts.
Here is an overview of some of the available
[3D Boolean operation and element cutting](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5) functionality.

**11)** We understand that is possible to create complex families based on the parametric values and formulas that take into consideration parameters. Using instance parameters we can manipulate the geometry of each instance of that family types – one simple example is this
[creation of a parametric rafter](http://www.revitzone.com/family-creation/107-creating-a-parametric-rafter).
Is it possible to add conditional rules such that some parts of the geometry will be generated or not according with some instance parameters values?
This is related to previous point.

**Answer:** Yes. Furthermore, retaining as much of this logic as possible in the family definition seems to be the more performant way to go right now, rather than adding void cuts etc. afterwards in the project.

**12)** Using the Revit API we try to create a new family, generate its geometry and load the new family into the project model using Document.LoadFamily(Document, IloadFamilyOptions). We obtain an Exception during the loading process. The same problem appears also in the SDK Sample FreeFormElement and in the BeamMaker project from your blog that do the same thing. We work in Visual Studio 2014. If we save the new family into the new RFA file and then we load it by reading the file, everything works very well. Are there any limitations there when manipulating new families?

**Answer:** Are you sure you are using the most current update release?
I encountered that issue myself and am pretty sure it has been resolved.
Even if not, all you have to do is save the RFA file and load the family from there, just as you say.
I implemented code to handle both situations to
[create and insert an extrusion family](http://thebuildingcoder.typepad.com/blog/2011/06/creating-and-inserting-an-extrusion-family.html).
I recently tested the FreeFormElement SDK sample again and did not observe any such issue.

**13)** How can I retrieve the relations between beams and architectural elements like walls, roofs, floors, to know which host object the beams belongs to? I understand that openings, doors, for example, have that relationship – the Host property of the openings returns the wall containing it.

**Answer:** There are a number of different ways to retrieve that relationship information, depending on various circumstances.
In a structural model, the structural analysis support information may return this.
For a generic solution, you can use ray casting or element filtering based on proximity, e.g. to
[filter for touching beams using solid intersection](http://thebuildingcoder.typepad.com/blog/2012/09/filter-for-touching-beams-using-solid-intersection.html) or
[determine the columns supporting a beam](http://thebuildingcoder.typepad.com/blog/2013/03/supporting-columns.html#2).

**14)** In case of beams if we create a beam in a work plane defined on the architectural element, will the HostFace property return a reference to the architectural element?

**Answer:** That is something you would have to try out for yourself in your own model and under your own specific circumstances.
The initial tool of choice for such exploration is RevitLookup and the BimChecker.
For more
[intimately interact with the model and API](http://thebuildingcoder.typepad.com/blog/2013/11/intimate-revit-database-exploration-with-the-python-shell.html) at
the same time, you could turn to the
[Revit Python Shell](http://code.google.com/p/revitpythonshell) or
[Revit Ruby Shell](https://github.com/hakonhc/RevitRubyShell).

**15)** Revit API we can add only shared parameters, and not project parameters like in Revit UI? Also for a family?

**Answer:** Correct.

**16)** Only one compound structure can be attached to a FootPrintRoof – is it possible to have different layer structure for each roof face?

**Answer:** As far as I know, the compound layer structure is associated with the roof type.
Therefore, all roofs of a given type will share the same layer structure.
If you would like it to vary, you will need to use different types for the different faces.

**17)** Can we create parts of a FootPrintRoof to obtain the layer geometry, similarly as you demonstrate to
[retrieve detailed wall layer geometry](http://thebuildingcoder.typepad.com/blog/2011/10/retrieving-detailed-wall-layer-geometry.html)?

**Answer:** I would assume that the slab can be separated into parts, just like a wall.

**18)** Do Ceiling and Floor elements have any significant structural difference?

**Answer:** Not that I am aware of.

**19)** Can a Ceiling element be created by API?

**Answer:** Not that I am aware of.

Thank you for these interesting questions.
Here are some more discussions related to performance handling large numbers of family instances and adding Boolean operations to them:

- [Performance Profiling](http://thebuildingcoder.typepad.com/blog/2010/03/performance-profiling.html)
- [Beam Maker Using a Void Extrusion to Cut](http://thebuildingcoder.typepad.com/blog/2010/07/beam-maker-using-a-void-extrusion-to-cut.html)
- [Boolean Operations and InstanceVoidCutUtils](http://thebuildingcoder.typepad.com/blog/2011/06/boolean-operations-and-instancevoidcututils.html)