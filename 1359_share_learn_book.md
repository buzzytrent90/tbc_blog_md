---
post_number: "1359"
title: "Share Learn Book"
slug: "share_learn_book"
author: "Jeremy Tammik"
tags: ['csharp', 'doors', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'python', 'references', 'revit-api', 'rooms', 'schedules', 'sheets', 'transactions', 'vbnet', 'views', 'walls', 'windows']
source_file: "1359_share_learn_book.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1359_share_learn_book.html"
---

### Sharing, Dynamo and a Chinese Book
I returned from the
[Autodesk Cloud Accelerator in Prague](http://autodeskcloudaccelerator.com/prague),
where I finished off cleaning up the
[FireRating in the Cloud](https://github.com/jeremytammik/FireRatingCloud) sample
and made some good inroads into the new
[CompHound project](https://github.com/CompHound/CompHound.github.io).
Today, I want to highlight some learning resources and sharing philosophy:
- [Håvard Vasshaug on Learning Dynamo and Sharing Content](#2)
- [Open Source BIM, IFC and FreeCAD](#3)
- [Chinese Revit API Book](#4)
- [Table of Contents](#5)
Before getting to that, here is my
[Cloud Accelerator Prague photo album](https://www.flickr.com/photos/jeremytammik/sets/72157656369939043) with
some pictures from last week:
[![Autodesk Cloud Accelerator Prague](https://farm6.staticflickr.com/5818/21435184406_b8b8b659c9_z.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157656369939043 "Autodesk Cloud Accelerator Prague")
#### Håvard Vasshaug on Learning Dynamo and Sharing Content
[Håvard Vasshaug](http://vasshaug.net/author/hvasshaug) published a really nice, professional and convincing in-depth discussion on
[Learning Dynamo](http://vasshaug.net/2015/09/18/learn-dynamo).
That led me to explore his site a teeny weeny bit more, for instance to discover the
[Vasshaug content libraries](http://vasshaug.net/content), including:
- Vasshaug Rebar Revit Library
- Dark Revit Content Library
- Flat Content by Andy Milburn and others. Downloaded, modified and standardized.
- Waffle Slab Void System for cutting Concrete Floors and obtaining a waffle slab.
- Steel Pile family with multiple voids for cutting concrete.
- Adaptive Component families for modelling Post Tension reinforcing bars.
There is some really valuable stuff in there.
I fully concur with Håvard's statement:
> I am inspired by sharing communities, and believe the international architecture, engineering and construction industry would do well to revise its visions about sharing knowledge and content. The idea that a company’s assets are files and folders, and that its employees will grow by keeping their knowledge to themselves is outdated at best.
Thank you very much for sharing all this, Håvard!
#### Open Source BIM, IFC and FreeCAD
On the topic of sharing, I also just noticed a whole host of open source BIM tools that I was unaware of:
- [BIMcollective](http://opensourcebim.org), a collective of open source developers that develop BIM software tools.
- [IfcOpenShell](http://ifcopenshell.org), the open source IFC toolkit and geometry engine
- [FreeCAD](http://www.freecadweb.org), a parametric 3D modeler, open-source, highly customizable, scriptable and extensible, multiplatfom (Windows, Mac and Linux), reads and writes open file formats such as STEP, IGES, STL, SVG, DXF, OBJ, IFC, DAE and many others.
I wish I had time to dive into it all.
#### Chinese Revit API Book
Finally, a very nice piece of news: a group of Chinese Autodesk engineers have published the first Chinese book on the Revit API. They received numerous customer questions about how to use the Revit API and how to use it effectively, igniting their passion to share their knowledge.
The book starts with a basic API introduction, then includes general platform API, RST/RME specific APIs, covers macros, etc.
It is now available. Here is a
[presentation of its publication](http://blog.csdn.net/lushibi/article/details/48653343) and a
[purchase link](https://detail.tmall.com/item.htm?_u=m1vm4lrf259d&id=521852354085).
The authors include:
- Lily, Tau, Elaine (Pangu team now)
- Janet (Nexus team now)
- Stephen (Michelangelo team now)
- Aaron (Dev Tech/ADN now)
The book is suitable for Revit API beginners, covering basics, allowing a novice to quickly learn the Revit API framework and proceed into Revit secondary development ranks. It covers all areas of architectural, structural, mechanical, electrical, family creation and Revit secondary development guidance.
It includes a lot of example code, pictures and tables to allow readers a better understanding. In a total of 15 chapters, it covers: function (event, interface, macros), class hierarchy (application class, a document class, elements, family, etc.), different Professional (architectural, structural, MEP related professional API), the development of Revit plug-ins for data read, create, modify, import, export and so on, API and .NET technologies to create a rich user interface, providing a better user experience, extending Revit itself, make Revit and other software platforms interact, data verification, inspection and automatic operation, improved data availability and efficiency of the design.
#### Table of Contents
1. Revit API Overview
1. Understanding Revit and Revit API
2. Revit API what you can do
3. Using the Revit API preparations
4. Online Resources
5. Development Tools
2. Revit API basics
1. External commands IExternalCommand and external applications IExternalApplication
2. Revit application class and document class (Application & Document)
3. Transaction processing (Transaction)
4. Practical example
3. Element (Element)
1. Element base
2. Element Edit
3. Filter element
4. Architectural Modeling
1. Elevation and Grids
2. Host element (HostObject)
3. Family instance (FamilyInstance)
4. Family instance (FamilyInstance) creation
5. Room and Area (Room and Area)
6. Line element (CurveElement)
7. Hole (Opening)
5. Notes (Documentation)
1. Dimensioning (Dimension)
2. Text annotation (Text)
3. Detailing (Detail)
4. Mark (Tag)
6. Geometry (Geometry)
1. Overview
2. Exercise: Get a wall of geometric data
3. Geometric primitive class
4. Geometry auxiliary class
5. Geometry collections
6. Exercise: Get a beam geometry data
7. Family (Family)
1. Introduction to Family
2. Related to the main API class
3. Management family type and family arguments
4. Management of the geometry
5. Visibility management of the geometry
6. Edit Family and Load Family
7. Other
8. View (Views)
1. Overview
2. Three-dimensional view (View3D)
3. Plan view (Plan View)
4. Drawing View (View Drafting)
5. Section View (View Section)
6. Reference callout view and detail view
7. Drawing View (Sheet)
8. Schedule (View Schedule)
9. Event (Events)
1. Introduction Event
2. Registration and cancellation of events
3. Cancellable event
4. Database Event
5. Interface events
6. Idle event (Idling Event)
7. External events (External Event)
10. Ribbon Extensibility (Ribbon UI)
1. Based on the introduction
2. Tab page (RibbonTab)
3. Panel (RibbonPanel)
4. Command button (PushButton)
5. Dropdown button (PulldownButton)
6. Dropdown memory buttons (SplitButton)
7. Drop-down combo box (ComboBox)
8. Drop-down combo box options (ComboBoxMember)
9. Select the button set and toggle buttons (RadioButtonGroup & ToggleButton)
10. Text box (TextBox)
11. Revit style task dialogs (Task Dialog)
11. Revit Structure Modeling
1. Structural model elements
2. Analysis Model (AnalyticalModel)
12. Material (Material)
1. Material Introduction
2. Identification materials
3. Patterning material information
4. Appearance information materials
5. Physical and heat information materials
6. Setting Materials
13. Plumbing Modeling
1. Duct / pipe (Duct / Pipe)
2. Electrical connections (Connector)
3. Plumbing model (MEPModel)
4. Plumbing system (MEPSystem)
5. Plumbing set up
6. Space and partitioning (Space & Zone)
14. Macro (Macro)
1. What is a macro
2. Revit Macros Introduction
3. Revit macros developed basic workflow
4. Modify and delete modules and macros
5. Run the macro in the Macro Manager
6. Debugging macros
7. Macro Security
8. Standard Revit API and the Revit macro uses the API differences
15. Other languages ​​(VB.NET, C++, CLI, F#)
1. VB.NET
2. C++ / CLI
3. F#
So much exciting stuff to look at!
Let's get going.