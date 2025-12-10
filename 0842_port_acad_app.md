---
post_number: "0842"
title: "Porting an AutoCAD Application"
slug: "port_acad_app"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'revit-api', 'walls']
source_file: "0842_port_acad_app.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0842_port_acad_app.html"
---

﻿

### Porting an AutoCAD Application

AutoCAD and Revit are
[very different animals](http://thebuildingcoder.typepad.com/blog/2012/02/bim-versus-free-geometry-and-product-training.html),
as we already pointed out a couple of times in the past.

Exaggerating just a little, you might say that Revit manages a BIM and ensures its consistency at all times, so there are no exceptions to anything ever, meaning nothing can be overridden.
Moreover, almost all Revit geometry is parametrically driven, so there is no way to tweak the model geometry either, except of course by changing the official parameter values.

This is obviously a challenge to developers with existing AutoCAD applications.
Here is another example of some typical considerations of an application developer with an existing full-blown AutoCAD application, wondering whether there is any way to start supporting Revit as well:

**Question:** We have customers using our software and asking if we could port our AutoCAD.NET application to Revit.

In our current implementation, we override line functionality to display special walls.
We need to modify subcomponents individually, manipulate grip points, split them, add more components, etc.

Intersections between walls affect neighbouring walls and their subcomponents.

We have therefore currently overridden grip points, snap points, graphics, properties, a custom ribbon is regenerated then an object is selected and so on.

Is it possible to implement something similar with Revit?
I guess we probably need to customize the functionality of some wall type?
What is best place to start if this is possible?

**Answer:** I am afraid that the answer is definitely not that you can simply port your existing application.

The Revit API is completely different from the AutoCAD one.

More than this, the entire underlying principles, workflows and paradigms are utterly different and almost beyond comparison.

In fact, one of the biggest stumbling blocks to learning the Revit API is being caught up in preconceptions coming from
[AutoCAD or other non-BIM APIs](http://thebuildingcoder.typepad.com/blog/2012/02/bim-versus-free-geometry-and-product-training.html).

In Revit, almost all the functionality you list above cannot be overridden, because Revit manages a building information model, BIM, and ensures that it remains consistent at all times.

The proper approach in Revit would probably be to define an appropriate family and family types.

I am not at all a product expert, though, so I cannot advise you at all about how best to implement your specific model in Revit.
You need to consult with application engineers and product experts about that, and above all understand the existing Revit functionality in depth before you even start considering creating an add-in application, which may easily end up reproducing or even fighting against the recommended Revit paradigms and workflows.

What I can tell you with a hundred percent certainty is that your existing AutoCAD application cannot simply be ported to Revit, and that your best approach will almost certainly require reconsidering the whole implementation from scratch.

What I can also say with conviction is that it will probably be well worth your while in the long run to start learning and understanding the Revit BIM concepts and researching how a useful add-in for that domain might look, to enable you to start work on a BIM-compatible implementation, in which you will certainly be able to reuse a lot of your existing know-how and continue supporting your customers as they gradually migrate to BIM usage.

If you have customers knowledgeable in Revit who are interested in cooperating with you to drive such a project, I would take that opportunity with great gratitude if I were you.

Best place to start? Learn the product functionality, and do your best to forget AutoCAD for a moment.

**Response:** Thank you for a great answer!

I think at this point we have to stick with AutoCAD due the limitations or the concept of Revit.
The main interest of our customer in using Revit was using IFC for integration with thermal calculation applications.

Of course Revit would also save us having to implement a number of additional objects like openings, roofs etc. that are not available in AutoCAD.

I believe we will continue implementing the functionality on top of AutoCAD but still continue to research Revit at the same time.
Perhaps at some point we could implement some kind of separate editor for our model, maybe using a WPF dialogue user interface that plays with normal walls in Revit.
If I read the API documentation correctly, we can still store additional info to wall objects.

**Answer:** Your approach sounds very feasible and sensible to me.

Implementing additional functionality in a separate WPF dialogue interacting with Revit would work fine.

Please be aware that you can also maintain your existing AutoCAD functionality and access that in the background, invisibly to the user, from your Revit add-in.

For example, you can read the relevant data from the Revit model, process it in the background using your existing application, and display useful results in your own WPF dialogue in a way that supports an efficient Revit workflow. That might enable you to achieve a near perfect solution with little effort.

And yes, absolutely, you can store any additional data you like on walls and any other elements using the
[extensible storage functionality](http://thebuildingcoder.typepad.com/blog/storage).

#### Importing SAT into Revit

People now and then ask about programmatically importing a SAT file into Revit.

You can export a SAT file using the API and import an SAT file from the user interface, simply by going through Insert > Import > Import CAD > ACIS SAT Files.

However, the Revit API Import methods only take options for DGN, DWG/DXF, gbXML, and images.

One obvious workaround is to implement an intermediate step to generate a DWG file from the SAT one.

This can be achieved either using RealDwg or AutoCAD itself, if it happens to be installed on the same machine.

In either case, the description of implementing
[ACISIN and ACISOUT in .NET](http://adndevblog.typepad.com/autocad/2012/05/acisin-and-acisout-in-net.html) might
come in handy to achieve this.

#### ElementTransformUtils and Family Instance Creation May Force Regeneration

If you are facing performance problems generating or manipulation largish models, please be aware that you need to watch out a bit with the ElementTransformUtils class.
Some of its methods contain built-in regeneration at the end, which is executed regardless of your other regeneration actions.
Family instance creation can also force regeneration at times.
As always, some research and lots of benchmarking is advisable.

#### AU Registration Savings End October 14

The end is near!

Oct 14 is the last chance to save $500 on the Autodesk University 2012 registration.

Prepare thyself and [register now](http://au.autodesk.com).

Hope to see you there!