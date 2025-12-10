---
post_number: "0972"
title: "Key Concepts of the Family Editor"
slug: "family_key_concept"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'geometry', 'levels', 'parameters', 'references', 'revit-api', 'rooms', 'schedules', 'selection', 'views', 'walls', 'windows']
source_file: "0972_family_key_concept.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0972_family_key_concept.html"
---

### Key Concepts of the Family Editor

Today I am returning back home from the very successful
[Revit API DevCamp in Moscow](http://www.autodesk.ru/adsk/servlet/pc/index?id=21516340&siteID=871736).

We received very good feedback from the participating developers who were so excited to learn about Revit programming that they ignored the hot environment, the unbearable sound of the false fire alarm and crazy running down the exterior staircase and back up again to the Autodesk offices on the 17th floor.

Here is Артур Кураков (Artur Kurakov) teaching one of the Revit API beginner sessions:

![](file:////j/photo/jeremy/2013/2013-06-25_moscow/p6240127.jpg)

I presented in three sessions, on the
[Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/03/revit-2014-api-and-room-plan-view-boundary-polygon-loops.html#2),
my
[2D round-trip cloud-based Revit model editor](http://thebuildingcoder.typepad.com/blog/2013/05/my-cloud-based-2d-editor-implementation-status.html),
and, together with my colleague Steven Campbell, Revit Content Project Manager and our resident Revit family expert, on the *Key Concepts of the Family Editor*.

Steve demonstrates the basics and some advanced aspects of the manual creation of a Revit family, key concepts that you as a Revit add-in developer absolutely must be aware of.

I had the honour and pleasure of helping out by implementing some sample add-in commands that I am rather happy with, showing how to drive families and their instances programmatically in the Revit project environment.
We'll get to those in a moment.

Let's start out with Steve's presentations:

- [Key family concepts](#2)
- [Building your first parametric Revit family](#2b)
- [Key family concepts slide deck bullets](#3)
- [Building your first parametric Revit family slide deck bullets](#3b)
- [Complete illustrated slide decks](#4)
- [Family API topics](#5)

Here are the two class descriptions, to define the context:

#### 1-4 – Key Concepts of the Family Editor – Steve Campbell (Стив Кэмпбелл)

Mastering Revit family creation is a key to the success when using Revit.
The same is true for Revit programmers.
The use of Family API is largely analogous to the UI.
Creating a family without fully understanding how Revit families work may lead to a bumpy road later on with your tasks.
Knowing how much you can get through proper use of families opens infinite possibilities.
This session offers an effective opportunity to learn directly from the family content expert, covering the following topics:

- Best practices
- Constraints – bone, muscle and skin analogy
- Types versus instances
- Family template selection

#### 1-5 – Building Your First Parametric Revit Family – Steve Campbell and Jeremy Tammik (Стив Кэмпбелл и Джереми Тэммик)

This is the continuation of the basic class on the key concepts of the family editor, presenting the following topics:

- Building your first parametric Revit family
- Driving nested families
- Programmatic use of families: place and modify instances, etc.

To give you a rough idea of the presentation contents, here is quick run-down of the slide deck bullet points:

#### Key Concepts of the Family Editor – Slide Deck Bullets

##### Families

What is a family?

A family is a group of elements with a common set of properties, called parameters, constraints, and a related graphical representations.

##### Types of Families:

System vs. Standalone vs. In Place

System Families:

Typically assemblies that are constructed on the construction site

Wall, Floors, Roofs, Ceiling, with some exceptions...

They only exist in a project or template file.

Standalone or "Loadable" Families:

Typically are manufactured pieces that are delivered to the construction site ready to install. They are intended to be used across many projects. Example: Windows, Doors, Light Fixtures, etc…

They are individual .rfa files.

In Place Families:

Project unique, built on site items that is stored only in the project.

##### Bone, Muscle and Skin:

So what is the best, most reliable way to build a family?

Build with the Bone, Muscle and Skin method.

Why?

Highest level of constraint is a labeled dimension attached to a reference plane.

The Bones, Muscle and Skin method describes the family as:

Bones = Reference Planes & Reference Lines

Muscle = Dimension, Label Parameters, Automatic Sketch Dimensions

Skin = Solids/Voids and Symbolic Linework

Blog Post by Steve Stafford:
[The Family Editor: Bones, Muscle & Skin](http://revitoped.blogspot.de/2009/04/family-editor-bones-muscle-skin.html)

##### The Bones: Workplanes

A work plane is a virtual 2-dimensional surface.

A work plane is used in the following ways:

- as the origin for a view
- for sketching elements on
- for placing work plane-based components on

Reference planes are the bones of a family

- They are a defined infinite plane in space
- They are intended to drive the geometric constraints

They have the added capability of defining the origin in a family, dimensioning to a family in a project, control snapping behavior and provide the availability of instance based shape handles.

Reference Lines are 3D lines with a start and end point that contains 4 workplanes.

They were designed to specifically for driving angular constraints.

##### The Muscles: Constraints and Parameters

##### Constraints

Constraints are a method of limiting or restricting an elements movement.

4 Types of constraints in Revit

- Dimensions
- Labeled Parameters
- Automatic Sketch Dimensions
- Pins

Constraints don't have to be static, they can be driven by a parameter value. Additionally a parameter’s value can also be driven by a formula.

##### Parameters:

A Parameter is a setting that determines the appearance or behavior of an element, type, or view.

Parameters store and communicate information about all elements in a model. Parameters are used to define and modify elements, as well as to communicate model information in tags and schedules.

Parameter Types:

3 basic types:

System, Family and Shared parameters…

System Parameters:

Are built in parameters defined in the software, they can't be removed or renamed, but they do schedule.

Note: when the parameter is selected in the Family Types dialog modify button is greyed out.

Family Parameters:

Are user based parameters that don't schedule.

Shared Parameters:

Are user based parameter with a shared definition and GUID that do schedule. The definition is stored in a shared parameter file, so the parameter can be loaded into many families.

##### Parameter Data:

Type Based Parameters:

Predefined size typical that can be found in a manufactures catalog.

Instance Based Parameters:

Variable based user input parameters used on items the can come in any length, like a wide flange beam is manufactured to any length the structure engineer requires. In this case the length of the beam is instance based.

Reporting Parameter:

Instance based parameter that can extract a value from a generic condition. This type of parameter can only be a length or angle.

##### The Skin: Elements

There are 4 types of elements:

- Model
- View specific
- View element
- Datum elements

Model Elements: (available in all views)

- Solids and Voids
- Extrusions, Blend, Revolve, Sweep and Swept Blend
- Model Lines

View Specific: (available in the views created in)

- Symbolic Lines
- Detail Lines

Each have element properties, for example:

Model Elements:

- Subcategory
- Material
- Visible
- Visibility/Graphics

These properties can be used to control how an element looks and is displayed, along with other properties of that element.

##### Where do I start?

##### Family Templates:

How do I decide which template to start from?

Don't select a template just based on the category, thinking about the functionality first will give you more options.

Functionality is not tied to the category but is tied to the template.

The category adds the family parameters, built in parameters and subcategories required for that category. Also this selection will effect how the family schedules.

Note: It does take some learning all the special cases in Revit.

Family Templates:

Decision tree

2D vs. 3D family?

2D – What is the use?

- Detail Item
- Profile
- Annotation
- Titleblock

3D – Does the family require specific functionality?

Yes – What is the special functionality for?

- Baluster
- Structural Framing
- Rebar
- Pattern Based

No – Does the family require a host?

Yes – Which host?

- Wall Based
- Ceiling Based
- Floor Based
- Roof Based
- Face Based

No – Then chose from:

- Standalone (Level Based)
- 2 Level Based (Column)
- Line Based
- Adaptive

Then choose a category.

Many of the family template are open to multiple categories giving you different functionality depending on your choices.

Example:

User wants to build an escalator

If they follow the category first, the user will have adjust the escalator height on placement and any time the floor to floor height changes.

If they follow the functionality first you can end up with a 2 level aware escalator that re-adjust its size automatically when the floor to floor height changes.

Template behaviors vs. category behaviors

Template have behaviors built in in relation to hosting, placement methods or even special editors

While categories have behaviors that are controlled by the settings in the Family Parameters under Family Category and Parameters dialog.

Changing the category changes the options available in the Family Parameters list. Some of the options include "Always Vertical", "Cut with Voids When Loaded" and "Shared"

Additionally setting the category changes the available built in sub-categories and parameters.

Visual example – Generic Model to Doors

##### The Process:

The “process” for building families is the most important aspect of family creation that one needs to learn.

##### Process order:

1. Pick your template
2. Plan (Insertion Point, Parametric Origin, etc.)
3. Layout Reference Planes (add The Bones)
4. Add Parameters and Constraints (add The Muscles)
5. Add multiple host thickness types (for testing hosted families)
6. Add 2 or more types
7. Flex Types and Host (Testing Procedure)
8. Add a Single Level of Geometry ( add The Skin)
9. Repeat until you are satisfied with the results
10. Test in Project Environment (create testing project)

##### Tips for Success:

A successful piece of content meets all the users needs for BIM.

Build not only for the user but how the information will be use along the whole BIM life cycle.

Ask yourself

- What is your use case? And how will it be used?
- What is the proper level of detail and or level of development for this piece of content?

Additionally try to meet all requirements for:

- Proper graphic representation in all view types.
- Enough information provided to support the down stream applications.
- Good parametrics for the expected use cases.
- Good performance

##### Additional Topics:

- Type Catalogs
- Is Reference

##### Type Catalogs:

Type Catalogs are an external group of types or catalog of a family in a delimited text format. Type catalog allow the user to selectively load a type from a long list.

The type catalog name must match the family name but with a .txt extension.

Tip:

The simplest way to create a type catalog is to export the current types by:

Click "R" > Export > (Family Types).

Open the file in a text editor and modify as needed.

```
,Manufacturer##other##,Length##length##centimeters,Width##length##centimeters, Height##length##centimeters
MA36x30,Revit,36.5,2.75,30
MA40x24,Revit,40.5,3.25,24
```

The first line defines the delimiter and the parameters in the type catalog.

The first character of the first line is the delimiter, in this case a comma ','.

The schema:

```
<parameter name>##<parameter type>##<units>,
```

The next lines are the catalog entries:

```
<type name1>,<parameter1>,<parameter2>,<parameter3>,<parameter4>
```

Parameter Declaration Table

##### Is Reference:

Reference planes have a property called “Is Reference”. By setting this property, you specify that the reference plane can be dimensioned or snapped to when you place a family into a project.

Additionally labeled instance parameters attached to reference planes with weak or strong “Is Reference” will allow shape handles for the user to control.

A “strong reference” has the highest priority for dimensioning and snapping.

For example, you create a window family and place it into a project. As you are placing the family, temporary dimensions snap to any strong references in the family. When you select the family in the project, temporary dimensions appear at the strong references. If you place a permanent dimension, the strong references in the window geometry highlight first. A strong reference takes precedence over a wall reference point (such as its centerline).

A “Weak reference” has the lowest priority for dimensioning and snapping. When you place the family into the project and dimension to it, you may need to press Tab to select a weak reference, as any strong references highlight first.

A “Not a reference” is not visible in the project environment so you cannot dimension or snap to those locations in a project.

All named “Is Reference”: Left, Right, Top… are considered strong references.

But the “Name” of the reference plane has no relation to the function of the “Is Reference” parameter.

#### Building Your First Parametric Revit Family – Slide Deck Bullets

##### Building Your First Parametric Revit Family

What are we going to do in this class?

1. Build a simple table

- Plan it first
- Build the Bones, Muscles and Skin

2. Drive Families from the API – Part 1

- Load
- Place
- Create new type
- Select
- Modify

3. Discuss and show Nesting in the Family Editor

- Parameter Linking
- Shared families
- Show examples

4. Drive Families from the API – Part 2

- Select all instances
- Modify nested family type

##### The Simple Table

Planning questions to answer:

1. Which template?

- M\_Generic Model.rft

2. Which category?

- Furniture

3. Placement?

- Origin to be centered Front/Back and Left/Right, on the Level

4. Parametrics?

- How to grow, equal Front/Back and Left/Right, up from Level

5. Special behaviour?

- None

6. Types to create?

- 1500x650x720mm
- 800x800x760mm

##### Process of building The Simple Table

Build in levels of detail or complexity using the Bones, Muscle and Skin methodology.

Process (short version):

- Layout Reference Planes (The Bones)
- Add Parameters and Constraints (The Muscles)
- Flex Types and Host (Testing Procedure)
- Add a Single Level of Geometry (The Skin)
- Flex Types and Host (Testing Procedure)
- Repeat for every level of detail or complexity
- Test in project environment

Order of Complexity:

- Table Top
- Legs
- Apron

##### Driving Families From API – Part 1

Table family:

- Load family
- Place instances
- Create new type
- Select instances
- Modify instance symbol

##### Nesting

Nesting enables the use of reusable parts in the family editor.

- The Parts can be driven from the host family.
- They can schedule separately (with limitations).
- They can be swappable.

Example:

- A swappable panel door family that is fully parametric.
- One family with 3 door panel families nested into the host.

##### Nesting Topics

Parameter Linking:

- Parameters can be linked from the host family to the nested families.
- Use the “Associate Family Parameter” button to link the 2 parameters.

Shared Families:

- Allow a family to be scheduled in a project.
- Limitation: Only instance based parameters can be linked.

##### Nesting Examples

- Open Web Joist that the web resize and array based on the depth and length
- Window grill pattern that can define number of spacing bars needed
- Kitchen cabinets with swappable door and drawer fronts
- Modular playground equipment

##### Driving Families From API – Part 2

Kitchen cabinets:

- Multiple select all instances
- Modify nested family type

##### Additional Resources

- Blogs

- [The Building Coder](http://thebuildingcoder.typepad.com)
- [Revit OpEd – Steve Stafford](http://revitoped.blogspot.com)
- [do-u-revit.blogspot.com](http://do-u-revit.blogspot.com)

- Newsgroups:

- AUGI – [www.augi.com](http://www.augi.com)
- [www.revitforum.org/forum.php](http://www.revitforum.org/forum.php)

- Books:

- [Paul Aubin](http://paulaubin.com)

#### Complete Slide Decks

In order to provide the associated images, here are the complete illustrated
[Key Family Concepts](file:////a/j/adn/devcamp/2013/doc/1-4_key_family_concepts.pdf) and
[Parametric Family](file:////a/j/adn/devcamp/2013/doc/1-5_5_parametric_family.pdf) slide decks.

#### Family API Topics

Steve's presentations above cover the key family editor concepts from a user point of view.

How to make use of this functionality programmatically is a large important topic that has been frequently discussed here in the past, e.g. looking at:

- [The Revit Family API](http://thebuildingcoder.typepad.com/blog/2009/08/the-revit-family-api.html)
- [Creating and inserting an extrusion family](http://thebuildingcoder.typepad.com/blog/2011/06/creating-and-inserting-an-extrusion-family.html)
- [All family API related posts](http://thebuildingcoder.typepad.com/blog/family)

To supplement the existing blog posts and Steve's product usage presentation for add-in developers, we decided to add new improved demonstrations of the following fundamental programming concepts to this overview:

1. Load family and place instances
2. Pick interaction and modify instances
3. Instance and symbol retrieval, nested type modification

This is where it gets really interesting for you, I hope.

Here is a snapshot of the current fully functional state of my
[FamilyApi](file:////a/j/adn/devcamp/2013/zip/FamilyApi09.zip) sample add-in
implementing the following three external commands providing a nice clean implementation of those concepts, plus an external application wrapper defining a compact user interface to access and launch them:

1. CmdTableLoadPlace
2. CmdTableNewTypeModify
3. CmdKitchenUpdate

The detailed explanation of these three commands is still forthcoming, hopefully tomorrow, so keep your eyes peeled.