---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 11.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.534236'
original_url: https://thebuildingcoder.typepad.com/blog/0199_family_api.html
post_number: 0199
reading_time_minutes: 15
series: family
slug: family_api
source_file: 0199_family_api.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- geometry
- levels
- parameters
- references
- revit-api
- schedules
- views
- walls
- windows
title: The Revit Family API
word_count: 3083
---

### The Revit Family API

Here is a rather lengthy and rich post to start off the week, describing the new family API introduced in Revit 2010 and including both an overview and in-depth information.

These are the main topics covered:

- [Creating a Family through the User Interface](#1).- [Creating a Family Programmatically](#2).- [Family API Samples in the Revit SDK](#3).- [Family API Labs: Creating an Example Family](#4).- [Family API Webcast and Materials](#5).

The text presented here is also a script for the Revit Family API slide deck which was used for the webcast mentioned at the end of this post.
If a picture says more than a thousand words to you, then you should download that slide deck, which will provide a couple of thousand words' worth of images to accompany this raw text.

##### Background

The concept of families is an enormous strength of Revit, but until Revit 2010, no programming access was available in the family context. Therefore, two large and completely disjunct developer communities have evolved around the Revit products, creating either:

- Revit applications using the API.
- Revit content with no API access.

The Family API was the
[top wish list item](http://thebuildingcoder.typepad.com/blog/2009/07/api-wish-list-survey-reminder.html)
and was made available for the first time in Revit 2010. It provides huge potential for synergy uniting the two separate camps. It enables:

- Use of the Revit API in the family editor.
- Extract and modify existing or create new family content.
- Automatic library generation.

#### Creating a Family through the User Interface

A non-trivial family can have a complex internal structure and many decisions need to be taken. A large body of experience around building families and family libraries has been developed before the introduction of the API. It is useful to gather some experience creating families manually before implementing code to do so automatically. Just like the standard Revit API, almost all the features provided by the family API are available through the user interface as well.

##### What is a Revit Family?

Before discussing the family API, it is important to understand the basics of Revit families and their definition.

A Revit family is a graphic representation of building objects and symbols. It can include geometry in 2D or 3D as well as data that supports the definition and creation of object instances. A family defines one or more types or symbols. A type or symbol can be inserted into the project to create a family instance.

There are three different classes of families, system, standard and in-place:

- System families are stored in the project template and used for objects such as Walls, Roofs, Floors, Ceilings, Rebar, etc.
- Standard families are defined externally in freestanding ".rfa" files and used for objects such as Windows, Doors, Furniture, Beams, Ductwork, etc.
- In-Place families are used for "one of kind objects".

The new family API provided in Revit 2010 addresses the standard families.

##### Revit Families - Where to Begin

A new family is always based on a family template file, if created from scratch, or on an existing family, which is enhanced in some way. Regardless of whether you are using the UI or API, the first thing you will need to decide is which template or family file you want to begin with.

- Create a completely new family starting from a family template.
- Enhance an existing family.

You need to choose which approach to take, as well as which template or family file to start with. There are plenty of templates to choose from. Other decisions that need to be taken and which influence this choice include

- Category?- Is the family 2D or 3D?- Model or detail component?- Hosted or non-hosted: Wall, Ceiling, etc.- Placement type: Free, two point...- Specialty: Lighting, RPC...

##### Revit Family Flavours

Just like the Revit product, Revit families also come in three flavours for architecture, MEP (mechanical, engineering, and plumbing), and structure. Most of the functionality is common to all three. Here are some aspects of the flavours:

Revit Architecture:

- Basic building components with simplistic interactions in the model.
- Free placement objects - casework, furniture, etc.- Two point placement objects - detail components, hosted objects.
  - Hosted objects: windows, doors, columns ("level to level"), ceiling or "wall based" lighting fixtures.

Revit Structure:

- Additional components with complex interactions with other objects.
- Framing - beams ("beams to beam", "beam to column"), columns.
- Trusses - layout for girder trusses; boundary conditions.
- Span direction symbols; reinforcement symbols - area reinforcement expands to find edges, path reinforcement.

Revit MEP:

- Connectors allowing objects to resize based on connected neighbour elements.

##### Revit Family Editor

Revit offers six basic family editors:

- 3D model, annotation, detail, rebar, truss and new conceptual mass.

Of the six basic family editors, the conceptual mass creation one is new to 2010. Depending on the editor, you will see a different set of available tools and building blocks. For instance, you will see tools to create forms in the model editor, but not in the annotation one. If you are using the truss editor, you will have access to the top and bottom chord, which will be shown in the model editor.

Each family editor is tied to the chosen family template and provides a specific feature set:

- Geometry - extrusions, blends, sweeps, revolves.
- Lines - model, symbolic, detail.
- Basic tools - copy, mirror, paint, join/unjoin, cut geometry/don't cut.
- References - reference planes, reference lines.
- Annotation tools - labels.
- Advanced tools - formulas, nesting, arrays, type catalogues.
- MEP tools - add connectors.

##### Revit Families Best Practice

Families are a powerful feature in Revit. Creating a family can be fun, and it can also be complex. When it becomes complex, it requires good planning. Here are some suggestions for a process for building families by the Autodesk Revit content manager Steve Campbell. It describes the manual definition of a family. The same applies to a programmatic approach as well. A key to understanding the family API is to understand the UI.

It is highly recommended to follow this structured process when building families. It needs to be learned and practiced. Systematically following this process is one of the most important aspects of family creation.

Process order:

1. Plan (insertion point, parametric origin).
2. Lay out reference planes (the bones).
3. Add parameters.
4. Add multiple host thickness types.
5. Add two or more types.
6. Flex types and host (testing procedure).
7. Add a single level of geometry.
8. Repeat steps 6 and 7 until you are satisfied with the results.
9. Test in project environment (create testing project).

##### Revit Family Possibilities

You can create quite complex objects and behaviour using Revit families. Here are a few of the possibilities:

- Formulas to control behaviour, visibility, arrays.
- Arrays and nesting for repeatable, resizable elements across an array.
- Advanced nesting with subcomponents that can be swapped.
- Reference lines and angular movement.

Formulas can be used to control behaviour, visibility, arrays, e.g. to define arrays of bolts depending on the size of a plate.

Arraying nested components allows the user to create families with repeatable elements across an array that can resize based on user input or rules. For example, a bookshelf with arrayed shelves, mullion patterns based on rules, and open web joists that adjust based on length and height.

Advanced nesting can make use of nested families with family type parameters that can provide flexible components with swappable sub-components such as nested door panels, frames, hardware, playground equipment, swappable panels and components.

Reference lines allow geometry to move about in an angular fashion. They contain two endpoints and two built in work planes that can be parametrically controlled. Some simple examples include a door swing that can change the opening angle, or a light fixture head that moves and points. A more complex example is an excavator arm that can bend and rotate about three or more pivot points.

#### Creating a Family Programmatically

Now that the basics of Revit families are clear, we can look at the new access to this functionality provided by the new family API.

##### Family API Usage

Exposure of the family API is probably the most important enhancement to the Revit API in 2010. The concept of component family is a unique feature and strength of Revit. This was the most wanted feature in the Revit API community and we expect the effect and growth in possibilities with the availability of family API will be dramatic.

An obvious opportunity provided by the new family API is the automatic generation of content from databases or other library sources.
It is also possible to extract a family definition out of a project and store it back into an external family file.
The document *Revit Platform API Changes and Additions.doc* in the Revit SDK folder provides an overview of the family API.
Family API specific samples are located in the Revit SDK samples `FamilyCreation` subfolder.

Here are some of the new supported features:

- Enable use of the Revit API within the family editor context.
- Create and modify family content.
- Automatic library generation from database or other library specification.
- Extract family definitions from existing projects.
- Define references and constraints to drive model geometry parametrically, formulas to drive parameter values, and annotation and dimensioning.
- Control detailed visibility of family types and their elements.
- Control loading behaviour of a family.

##### Document and Family Manager Classes

The Revit API Document now has some added methods and properties for managing families:

- EditFamily - edit a family loaded in a project document.
- FamilyCreate - return a FamilyItemCreate object to create new instances of elements within a family document, analogous to the Create object in a project.
- FamilyManager - return a FamilyManager object providing access to family types and parameters.
- IsFamilyDocument - identify whether the current document is a family document.
- OwnerFamily - return the owning family of this family document.

Within a family document, the family manager class provides the following new functionality:

- Add, remove and rename types.
- Add and remove parameters.
- Set values and formulas.

##### Creating Family Content

The FamilyCreate property on the family document returns a FamilyItemFactory instance. This family item factory object is a utility object used to create new instances of elements within the family document. Just like other Revit elements, these are instantiated using dedicated methods instead of the .NET new operator. This ensures that the elements created are correctly added to and hooked up within the family document. A wide range of elements types can be created, including alignments, dimensioning, annotation, curves, levels, and solid forms for conceptual design.

The family item factory utility object

- Creates new instances of elements within the family document.
- Provides dedicated creation methods instead of the .NET new operator.

Elements types that can be created include:

- Alignment.
- Connector (MEP only).
- Dimensioning.
- Annotation.
- Curves.
- Levels.
- Solids forms.

##### Visibility Settings

A critical topic when building family content are the visibility settings. They are now accessible for each element in a family through the new FamilyElementVisibility class.

Each element in a family has its own visibility settings which define which levels of detail and which types of views it appears in. These options are critical to building good content. For example, intricate details of a family should only be visible in the fine detail views. 3D solid content could optionally be suppressed in plan views, where light weight 2D line work could be displayed instead. Such an approach can make a substantial performance difference, especially in large building models.

Every element in the family has own visibility settings managed by the FamilyElementVisibility class defining:

- Which levels of detail it appears in.
- Which types of views it appears in.

##### Loading Control

The Document.LoadFamily method has been enhanced and new overloads have been added, which can help to handle situations such as when a family already exists in the project. The following overloads are now provided:

- LoadFamily(Document) - loads the contents of this family document into another document.
- LoadFamily(String) - loads an entire family and all its types into the document.
- LoadFamily(String, Family) - loads an entire family and all its types into the document and provides a reference to the loaded family.
- LoadFamily(Document, IFamilyLoadOptions) - loads the contents of this family document into another document.

The IFamilyLoadOptions argument to the last overload defines an interface which specifies two call-backs for handling family load situations: OnFamilyFound and OnSharedFamilyFound. These are called when a family or a shared family is already present in the target document.

#### Family API Samples in the Revit SDK

A number of new samples illustrating the family API have been added to the Revit SDK. They are located in the FamilyCreation subfolder in the SDK Samples directory.

##### AutoJoin

AutoJoin automatically joins geometry of multiple generic forms for use in family modelling and massing. It uses the method Document.CombineElements to join geometry between overlapping generic forms. It also includes a utility method to check geometry object overlap, based on the Face.Intersect(Curve) method.

##### AutoParameter

AutoParameter implements batch mode automatic addition of shared or non-shared parameters to one or more family documents. It optionally processes either the currently active family document or all families in a specified folder. It uses the FamilyManager class AddParameter methods and reads its input data from parameter text files in a format similar to the Revit shared parameter files.

##### CreateAirHandler - RME

CreateAirHandler is a Revit MEP sample to create an air handling unit including MEP pipe and duct connectors. It shows how to check the template family category to verify that a valid starting point is selected. It make use of the FamilyItemFactory class NewExtrusion, NewPipeConnector, and NewDuctConnector methods, sets up the proper connector parameters, and uses Document.CombineElements to join the extrusions to make up the air handler body.

##### CreateTruss - RST

CreateTruss is a Revit Structure sample that creates a mono truss in a truss family document. The truss curves are created using NewModelCurve, the truss type is set through the ModelCurve TrussCurveType property, and constraints are added to the truss curves with NewAlignment.

##### DWGFamilyCreation

DWGFamilyCreation shows how to import a DWG file into a family document add two type parameters to the imported instance: DWGFileName specifying the DWG file name, and ImportTime storing the data and time when it was imported.

##### GenericModelCreation

GenericModelCreation creates a generic model using extrusion, blend, revolution, sweep and swept blend elements. It checks that the open document is indeed a family one or otherwise creates a new family document. It exercises the CreateSketchPlane, NewLineBound, and FamilyItemFactory methods to create profiles and shapes.

##### TypeRegeneration

The TypeRegeneration sample uses the FamilyManager Types property to determine all types defined in the current family document, and CurrentType to iterate through them. It reports whether all types regenerated successfully, and logs any errors that occurred to a file.

##### ValidateParameters

ValidateParameters checks whether every type in the current family document has valid values for certain parameters and logs the results to a file. This sample can be run in two modes, either as an external application subscribing to DocumentSaving and DocumentSavingAs events to run the check automatically every time a document is opened, or as an external command to be launched manually when required.

##### WindowWizard

WindowWizard shows how to create a new window family via a wizard style user interface. It needs to be started in a window family template, e.g. Metric Window.rft. It prompts the user to define input dimensions for various window parameters and materials, and then creates the required geometry, constraints and types using elements including extrusions, alignments, dimensions, reference planes, and family types.

#### Family API Labs: Creating an Example Family

The Revit Family API Labs is a collection of exercises which introduce you step by step to the creation of a column family. The objective is to learn the basics of the family API. The labs start with the basics and then proceed to more advanced aspects. Full documentation of and instructions for each step are included in separate documents for C# and VB.

##### Family API Labs

- Lab1 - define a column with rectangular profile.
- Lab2 - define a column with L-shape profile.
- Lab3 - add formula and materials.
- Lab4 - add visibility control.

##### Lab 1 - Create Rectangular Column

- Check the family context.
- Create a simple solid using extrusion.
- Set alignments.
- Add types.

Classes and methods:

```
doc.IsFamilyDocument()
doc.OwnerFamily.FamilyCategory.Name
doc.FamilyCreate.NewExtrusion()
doc.FamilyCreate.NewAlignment()
familyMgr = doc.FamilyManager
familyMgr.NewType()
familyMgr.Parameter(); familyMgr.Set()
```

##### Lab 2 - Create L-Shaped Column

- Add reference planes.
- Add parameters.
- Add dimensions.

Classes and methods:

```
doc.FamilyCreate.NewReferencePlane()
familyMgr.AddParameter()
doc.FamilyCreate.NewDimension()
```

##### Lab 3 - Add Formulas and Materials

- Add formulae.
- Add materials.

Classes and methods:

```
familyMgr.SetFormula()
pSolid.Parameter("Material")
familyMgr.AddParameter("MyColumnFinish", BuiltInParameterGroup.PG_MATERIALS, ParameterType.Material, True)
familyMgr.AssociateElementParameterToFamilyParameter()
```

##### Lab 4 - Add Visibility Control

- Add line representation.
- Add visibility control.

Classes and methods:

```
doc.FamilyCreate.NewSymbolicCurve()
doc.FamilyCreate.NewModelCurve()
FamilyElementVisibility(FamilyElementVisibilityType.ViewSpecific/Model)
FamilyElementVisibility.IsShownInFine, etc.
pLine.SetVisibility(pFamilyElementVisibility)
```

##### Learning More

Here is an overview of some available resources for further learning:

- Online Help, Developer's Guide and SDK Samples- Families Guide

    <http://usa.autodesk.com/adsk/servlet/item?siteID=123112&id=13376394>
  - DevTV Introduction to Revit Programming

    <http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=2484975>
  - Recording of Revit 2010 Programming Introduction Webcast

    <http://www.adskconsulting.com/adn/cs/api_course_sched.php> > Revit API
  - Discussion Group

    <http://discussion.autodesk.com> > Revit Architecture > Revit API
  - API Training Classes

    <http://www.autodesk.com/apitraining>
  - The Building Coder, Jeremy Tammik's Revit API Blog

    <http://thebuildingcoder.typepad.com>
  - Autodesk Developer Network

    <http://www.autodesk.com/joinadn>
  - DevHelp Online for ADN members

    <http://adn.autodesk.com>

#### Family API Webcast and Materials

We held a successful
[webcast](http://thebuildingcoder.typepad.com/blog/2009/07/revit-family-api-webcast.html)
on the Revit Family API on Thursday July 23rd 2009.
The supporting material and recording from this webcast is now available.
To access the material from any ADN webcast, you can go to the
[ADN training schedule site](http://www.adskconsulting.com/adn/cs/api_course_sched.php)
and click on the corresponding download link.
You can find this specific webcast by filtering for 'Revit Family API'.
Here is a direct link to
[download the webcast material](http://download.autodesk.com/media/adn/Revit_2010_Family_API_Webcast-July2009.zip), which includes the following items:

- **Presentation:** Revit 2010 Family\_API.pptx.- **Source Code for Labs:** Revit Family Labs including Visual Studio solution file, Revit.ini information, C# and VB source code, instructions, and a Revit RFT template file.- **Recording:** Revit Family API Webcast recording.- **Q & A:** Revit 2010 Family API Webcast - Questions and Answers Jul 23 2009.docx.