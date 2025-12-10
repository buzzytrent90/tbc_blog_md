---
post_number: "0117"
title: "Revit Structure Resources"
slug: "rst_resources"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'geometry', 'parameters', 'revit-api', 'rooms', 'schedules', 'sheets', 'views', 'walls', 'windows']
source_file: "0117_rst_resources.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0117_rst_resources.html"
---

### Revit Structure Resources

**Question:**
What resources are available for getting started with the Revit Structure API, especially concerning the implementation of an analysis link, a link between Revit Structure and an external analysis application?

**Answer:**
There are a number of resources available on the Revit Structure or RST API, and we from the Autodesk DevTech team have held several trainings, webcasts and classes at Autodesk University on this topic.
The Revit Structure training material and samples from these trainings is available, some of it publicly and some on the ADN web site. Here are the main resources that I am aware of:

- [Revit Structure training material, samples and webcast recording](#adn)
- [Autodesk University class material](#au)
- [MidasLink analysis link sample application](#midaslink)

There are also a number of RST specific samples in the standard Revit SDK.
Let's look at the contents of each of these packages in more detail.

#### Revit Structure Training Material and Samples

The RST training material has focused on two main areas:

- Analysis link
- Rebar and detailing

These correspond to and aim to help support and automate the tasks and user focus not addressed very strongly by the base RST product:

- Structural analysis by engineers and designers early on in the design phase.
- Creation of fabrication and shop drawings for the contractor and steel detailer in the fabrication phase.

![RST API focus](img/rst_api_focus.gif)

The samples provided in the RST training material consist of the following components:

- RST labs: a collection of labs demonstrating how to read and write RST elements and data relevant to the analysis link.
- RST link: a sample application simulating an extremely rudimentary link between RST and an external analysis application.
- Rebar and detailing samples: enhancements to add additional RST rebar and detailing oriented functionality to some standard SDK samples.

The RST labs explore the Revit elements and data involved in data exchange between RST and an external analysis application and demonstrate how they are related and can be accessed, modified, and created.
In a second step, the RST link sample makes use of the labs functionality to implement a simulated analysis link. AutoCAD is used to simulate the external analysis application. The data is exchanged between RST and AutoCAD using a custom XML format. Some RST properties are exposed in AutoCAD and stored in extended entity data. They can be edited manually in the AutoCAD Object Property Manager or OPM, and the updated data is read back in to Revit. Currently, only 'stick' elements are supported, i.e. structural framing elements such as columns, beams and braces. Here is an overview of the different RstLink modules:

- RstLink: Helper DLL shared by both AutoCAD and Revit client.
- RSLinkRevitClient: Revit command implementation defining the two commands RSLinkImport and RSLinkExport.
- RSLinkRevitApp: Revit external application defining a user interface for the two commands.
- RSLinkAcadClient: AutoCAD.NET client defining the commands RSImport, RSExport and RSMakeMember.
- RSLinkAcadClientDynProps: Dynamic Revit properties for AutoCAD objects.

The enhanced detailing samples demonstrate:

- Generation of a section and drafting view
- Creation of a sheet
- Import and export of external file formats
- Adding text, dimensioning and annotations

The enhanced standard Revit SDK samples are included in some of the the standard Revit SDK distribution samples:

- CreateDimensions
- CreateViewSection: CreateDraftingView
- TagBeam: TagRebar, CreateText

Here is a list of the sample commands provided, extracted from my ADN samples text file, which is processed by RvtSamples to define entry points in the Revit 2009 menu:

#### Analysis link commands

- 1-1 Load Natures, Cases and Combinations: List load grouping objects, i.e. load natures, cases, and combinations
- 1-2 Load Objects - List all : demonstrate access to load objects and list point, line and area loads
- 1-3 Load Objects - Modify selected: more detailed info and modification of selected loads
- 1-4 Load Symbols: list load symbols
- 1-5 Create New Point Loads
- 2-1 Structural Columns: list structural columns
- 2-2 Structural Framing: list structural framing elements
- 2-3 Structural Foundations: list structural foundation elements
- 2-4 Structural Standard Family Instances: list standard family instances with an analytical model
- 2-5 Structural System Family Instances: retrieve structural system family instances: wall, floor, continuous footing
- 3 Analytical Model: list analytical model for selected or all structural elements
- RstLink Export: export to RstLink file (from Revit to AutoCAD)
- RstLink Import: import from RstLink result file (from AutoCAD to Revit)

#### Standard SDK and enhanced rebar and detailing commands

- Analytical Viewer:

  Draws analytical model for selected or all elements in the viewer window

  AnalyticalViewer- Create Beams Columns and Braces

    Create configurable multi-story Beams Columns and Braces using simple dialog

    CreateBeamsColumnsBraces- Rebar - Create Reinforcement

      Create bar set in a selected concrete element (beam or column) that does not have any reinforcement.

      Reinforcement- Rebar - Edit AreaReinforcement Parameters

        Show parameters of selected AreaReinforcement and allow user to modify

        AreaReinParameters- Rebar - List Rebar Parameters

          Show parameters of selected AreaReinforcement and allow user to modify

          AreaReinParameters- Rebar - Create Section View

            Create a section view across the midpoint of the selected wall, floor or beam

            CreateViewSection- Rebar - Create Drafting View

              Create a new empty drafting view

              CreateViewSection, CreateDraftingView- Rebar - Import And Export DWG

                Export current project to dwg files and import a dwg file into revit

                ImportExport- Rebar - Create Dimension

                  Add a dimension to a selected structure wall from its start to its end

                  CreateDimensions- Rebar - Tag Beam

                    Tag beam's start and end

                    TagBeam- Rebar - Tag Rebar

                      Tag selected Rebar

                      TagBeam TagRebar- Rebar - Create Text

                        Create a new TextNote instance for selected Rebar

                        TagBeam CreateText

This material was presented in various classroom trainings and webcasts.
A Revit Structure API webcast was held in May 2008.
The material and recordings of all ADN webcasts are publicly available from the
[Developer Center training site](http://www.adskconsulting.com/adn/cs/api_course_sched.php)
listing the course schedule for all our trainings and webcasts.
Here is a
[direct link](http://download.autodesk.com/media/adn/RevitStructure2009APIWebcast_13May08.zip)
to the
[materials and recording](http://download.autodesk.com/media/adn/RevitStructure2009APIWebcast_13May08.zip)
for this webcast.
For your convenience, I am making the presentation and sample code available right here as well:

- [Powerpoint presentation](zip/rst_api_ppt_20090401.zip)
- [RST labs sample code](zip/rst_labs_20090401.zip)
- [RST link sample code](zip/rst_link_20090401.zip)

#### Autodesk University Class Material

We held classes on the RST API at Autodesk University in 2007 and 2008.
In 2007, there were two separate sessions on the analysis link and the rebar and detailing topics:

- [DE201-1 Create Your Own Bidirectional Revit Structure Stress Analysis Integration Link Revit Structure](http://au.autodesk.com/?nd=class&session_id=1048)
- [DE205-1 Reinforce Your Design: Revit Structure API for Rebar and Detailing Revit Structure](http://au.autodesk.com/?nd=class&session_id=1132)

In 2008, they were combined into one single streamlined class held by Mikako Harada:

- [DE215-3 Revit Structure API: From Bi-Directional Link to Rebar Detailing](http://au.autodesk.com/?nd=class&session_id=3136)

This latter session was also recorded.
Here is the class description and some highlights:

Two main areas of interest in the Revit Structure (RST) API are the bi-directional link to an external structural
analysis program and reinforced concrete detailing design. In addition to the physical building information model
(BIM) defined in Revit Architecture, RST defines an analytical model of the building, composed of geometry,
loads, connectivity, release/boundary conditions, material properties, and other project parameters. This session
shows how to implement a bi-directional link to a structural analysis program. We will then look at the
reinforcement concrete detailing API and show how to automate part of the reinforcement workflow, eliminating
repetitive manual tasks. This session assumes basic knowledge of Revit programming.

- Implement a Revit Structure add-in to export and import between the RST analytical model and an external analysis program.
- Implement an add-in that automates repetitive manual tasks in production of construction documents.
- Analyze, modify and create building model geometry, loads, connectivity, release and boundary conditions.
- Automatically drive the framing modelling, generate rebar, and extract rebar information.
- Programmatically generate section and drafting views and import and export DWG files, such as schedules.
- Create RST macros to add text, dimensions, and annotations.

The
[class material](http://au.autodesk.com/?nd=class&session_id=3136)
is publicly available from the Autodesk University web site and includes:

- Handout document
- Powerpoint presentation
- Code samples
- Recording

#### MidasLink Analysis Link Sample Application

For completeness sake, I also mention MidasLink.
This is the result of a consulting project linking to a commercial application.
It is only accessible to ADN members.

MidasLink is a Revit Structure add-in program that exports and imports the Revit model to and from the MIDAS/Gen structural analysis application.
The source code is provided to help developers integrating analysis programs with Revit Structure.
The compiled add-in is available to Revit Structure subscription customers in certain countries.
Source code for the 2008 and 2009 versions are available on the ADN web site.
To find it, you can simply search for 'MidasLink'.
Here is a
[direct link](http://adn.autodesk.com/adn/servlet/item?siteID=4814862&id=11816840&linkID=4901650) to the 2009 version.
The package includes:

- Analysis application executable
- Documentation
- Source code
- Installer

The installer source code is also included, which is in C++.
This makes it interesting for any Revit application developer to have a look at.
Another module of interest to any developer is the unit handling, which I already mentioned in a
[previous post](http://thebuildingcoder.typepad.com/blog/2008/09/units.html).