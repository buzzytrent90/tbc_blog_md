---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:50.870137'
original_url: https://thebuildingcoder.typepad.com/blog/0901_whats_new_2010.html
parent_post: 0901_whats_new_2010.md
part_number: '01'
part_total: '42'
post_number: 0901
series: whats_new_in_revit_api
slug: whats_new_2010_parameters
source_file: 0901_whats_new_2010.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- parameters
- references
- revit-api
- rooms
- schedules
- views
- walls
title: What's New in Revit 2010 API - Parameters
word_count: 1330
---

﻿

# What's New in the Revit 2010 API

I am in vacation in Italy, and trying not to work.

There is one project that I had in mind for quite a while, though, and will start addressing now anyway during this break: I plan to publish the contents of the Revit API documentation 'What's New' sections from the previous few releases of the Revit API help file RevitAPI.chm here, so that they become included in future Internet searches.

The reason for this is that I have found them to be a very valuable source of information.
I regularly look for answers in these document sections myself first of all when I encounter a question in an API area that I am not familiar with.

So let's start chronologically, with the Revit 2010 API news.

Well, on second thoughts, the Revit 2008 and 2009 API releases also included some 'What's New' information in the form of an 'API Changes' folder containing XML files listing the additions and removals, which I will simply provide for download right here:

- [Revit 2008 API Changes](zip/2008_API_Changes.zip)
- [Revit 2009 API Changes](zip/2009_API_Changes.zip)

For Revit the 2010 API, I actually extracted the following from the document 'Revit Platform API Changes and Additions.doc' provided in the Revit 2010 SDK, which lists the same information as the help file.

I plan to follow up with the corresponding information for the Revit
[2011](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2011-api.html),
[2012](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2012-api.html) and
[2013](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html) APIs
by and by.

# Major Enhancements and Modifications Affecting the Revit 2010 API

## New Revit user interface

Autodesk Revit 2010 offers a new user-interface system based on Ribbons. The API has been modified to support customization of the Ribbon user-interface in a similar manner to what was supported in previous releases for toolbars and menus.

External commands registered directly to Revit via the Revit.ini file are now located on the “Add-ins” tab under the “External commands” pulldown. No code changes are required to move the buttons to this location. The Add-ins tab will be automatically displayed if any external command is registered with Revit.

External applications can register and lay out panels on the Add-ins tab of the Ribbon. Custom panels may be given an application-specified name, and may contain:

- Pushbuttons – a single button with label and icon that invokes a command when pressed.- Pulldown buttons – a button with label and icon that opens a menu with different choices when pressed.- Separators – a vertical line separating panel components.

Pushbuttons and Pulldown buttons may be created as large size, or stacked in sets of 2 or 3.

The API for creating Ribbon Panels is completely different from the Toolbar and Menubar API. External applications which added content to the toolbars and menus will need to be modified to create Ribbon panel(s) instead.

## Family Creation API

The Family Creation API provides programmatic access for creation and modification of the contents of Revit Family documents. Geometric data, Family Types, and other information that can be specified through the Revit Family Editor can now be specified via the API. As a result of this project, API external commands are now displayed while editing a family document in the Family Editor. API access to in-place family editing is still restricted. The project introduces the class Autodesk.Revit.FamilyManager, which allows access to family types and parameters. Using this class you can add and remove types, add and remove family and shared parameters, set the value for parameters in different family types, and define formulas to drive parameter values.

The Category and Categories classes have been enhanced and a new GraphicsStyle class has been created to provide control of the appearance of Revit families.

The new class FamilyItemFactory provides the ability to create elements in families. Access this class through the Document.FamilyCreate property. It includes the following methods:

- NewBlend- NewDimension- NewDuctConnector (in Revit MEP only)- NewElectricalConnector (in Revit MEP only)- NewExtrusion- NewFamilyInstance- NewModelText- NewPipeConnector (in Revit MEP only)- NewRevolution- NewSketchPlane- NewSweep- NewSweptBlend- NewSymbolicCurve- NewTextNote

The new class ItemFactoryBase provides the ability to create elements both in project and family documents and includes the NewAlignment method.

New document methods – Document.LoadFamily and Document.EditFamily – are introduced to bring families from the Family Editor to the Project environment and vice versa. These methods can be used to examine the contents of the family in terms of its elements, parameters and types; as a result, the properties of Family which access the contents have been removed (Family.SolidForms, Family.VoidForms, Family.Components, Family.LoadedSymbols, Family.Others).

## New Events

The Revit API offers totally revamped events that allow an application to subscribe to events taking place in the Revit user interface or in API workflows. In all of the new events, events now occur before and after various triggering actions (known as "pre" and "post" events). Many of the new pre-events are cancellable, offering the API application the ability to prevent the event from taking place based on criteria it cares about. (It is the responsibility of the event subscriber to inform the user of the reason for the cancellation).

Events that existed before release 2010 have been marked Obsolete. You should use the new events added in this release to cover situations like:

1. Document creation- Document opening- Document saving- Document saving as (new filename)- Document closing- Dialog box showing

The dialog box showing event has been restructured to support notification by Revit when task dialogs are shown.

The new events added in Revit 2010 are:

1. File export- File import- View printing- Document printing- View activating- Document synchronize with central

## API for new Massing functionality

Revit 2010 introduces new conceptual design functionality for creation of complex geometry. Intuitive and flexible form-making is supported by the addition of new objects: points and spline curves that pass through these points. The resulting surfaces can be divided, patterned, and panelized to create buildable forms with persistent parametric relationships.

The following classes provide the API user with access to this functionality:

- CurveByPoints- CurveByPointsArray- DividedSurface- DividedSurfaceData- Form- ReferencePoint- ReferencePointArray- TilePatterns- TilePattern

Other methods and properties added to support this functionality are:

- ModelCurve.ChangeToReferenceLine()- FamilyCreate.NewLevel(double)- Family.IsCurtainPanelFamily- Family.CurtainPanelHorizontalSpacing- Family.CurtainPanelVerticalSpacing- Family.CurtainPanelTilePattern- SketchPlane.PlaneReference- Level.PlaneReference

The following classes cannot be used in 2010 Mass families. Instead, equivalent geometry can be created with the Form class.

- Blend- Extrusion- Revolution- Sweep- SweptBlend

## MEP API for ducts and pipes

This project provides read/write access to HVAC and Piping data in a Revit model including:

1. Traversing ducts, pipes, fittings, and connectors in a system- Adding, removing, and changing ducts, pipes, and other equipment- Getting and setting system properties- Determining if the system is well-connected

The following classes have new methods and properties related to duct and pipes:

- Autodesk.Revit.MEP.Connector- Autodesk.Revit.Elements.MechanicalSystem- Autodesk.Revit.Elements.PipingSystem

Elements related to duct and pipe systems can be created using new methods on Autodesk.Revit.Creation.Document:

- NewCrossFitting- NewDuct- NewElbowFitting- NewFlexDuct- NewFlexPipe- NewMechanicalSystem- NewPipe- NewPipingSystem- NewTakeoffFitting- NewTeeFitting- NewTransitionFitting- NewUnionFitting

All MEP-related classes have been moved into the MEP namespace from namespaces such as Elements and Symbols.

## Upgrade to .NET Framework version 3.5

Revit now uses Microsoft .NET Framework version 3.5. Applications compiled using .NET 2.0 should continue to work unless they are affected by other changes in the Revit 2010 API.

# Small enhancements & API interface changes

### Parameters

The new property Autodesk.Revit.Parameters.InternalDefinition.BuiltInParameter provides access to the enumerated type for the parameter, if it is a built-in parameter. The property makes it easier to locate a built-in parameter value based on the name of the parameter shown in the Revit UI.
---

## Related Sections
- [What's New in Revit 2010 API - Rooms and Spaces](./0901-02_rooms_and_spaces.md)
- [What's New in Revit 2010 API - Grids](./0901-03_grids.md)
- [What's New in Revit 2010 API - Batch creation methods (NewWalls, NewGrids, etc.)](./0901-04_batch_creation_methods_newwalls_newgrids_etc.md)
- [What's New in Revit 2010 API - Rebar](./0901-05_rebar.md)
- [What's New in Revit 2010 API - Structural framing](./0901-06_structural_framing.md)
- [What's New in Revit 2010 API - Categories](./0901-07_categories.md)
- [What's New in Revit 2010 API - Sketch](./0901-08_sketch.md)
- [What's New in Revit 2010 API - Revit session properties](./0901-09_revit_session_properties.md)
- [What's New in Revit 2010 API - External Commands and Applications](./0901-10_external_commands_and_applications.md)
- [What's New in Revit 2010 API - HostedSweep](./0901-11_hostedsweep.md)
- [What's New in Revit 2010 API - VolumeCalculationOptions](./0901-12_volumecalculationoptions.md)
- [What's New in Revit 2010 API - Curtain Systems](./0901-13_curtain_systems.md)
- [What's New in Revit 2010 API - Output Arguments from API methods](./0901-14_output_arguments_from_api_methods.md)
- [What's New in Revit 2010 API - Visual Studio Tools for Applications (VSTA) 2.0](./0901-15_visual_studio_tools_for_applications_vsta_20.md)
- [What's New in Revit 2010 API - Find references in the direction of a ray](./0901-16_find_references_in_the_direction_of_a_ray.md)
- [What's New in Revit 2010 API - Reference parsing](./0901-17_reference_parsing.md)
- [What's New in Revit 2010 API - Element Hiding](./0901-18_element_hiding.md)
- [What's New in Revit 2010 API - Parameter.Set return value when value is unchanged](./0901-19_parameterset_return_value_when_value_is_unchanged.md)
- [What's New in Revit 2010 API - NoConditionType removed for Spaces](./0901-20_noconditiontype_removed_for_spaces.md)
- [What's New in Revit 2010 API - Detail Curves use sketch plane of their view](./0901-21_detail_curves_use_sketch_plane_of_their_view.md)
- [What's New in Revit 2010 API - SketchPlane property added to View class](./0901-22_sketchplane_property_added_to_view_class.md)
- [What's New in Revit 2010 API - Export family data to XML for Autodesk Seek integration](./0901-23_export_family_data_to_xml_for_autodesk_seek_integr.md)
- [What's New in Revit 2010 API - Exceptions when parameters are disabled](./0901-24_exceptions_when_parameters_are_disabled.md)
- [What's New in Revit 2010 API - New options for export to DWF](./0901-25_new_options_for_export_to_dwf.md)
- [What's New in Revit 2010 API - New option for export to FBX](./0901-26_new_option_for_export_to_fbx.md)
- [What's New in Revit 2010 API - New method for export to DGN](./0901-27_new_method_for_export_to_dgn.md)
- [What's New in Revit 2010 API - New method for export to Civil3D](./0901-28_new_method_for_export_to_civil3d.md)
- [What's New in Revit 2010 API - New method for import from gbXML](./0901-29_new_method_for_import_from_gbxml.md)
- [What's New in Revit 2010 API - New method for import from Inventor](./0901-30_new_method_for_import_from_inventor.md)
- [What's New in Revit 2010 API - Document.Application property](./0901-31_documentapplication_property.md)
- [What's New in Revit 2010 API - New element collector properties](./0901-32_new_element_collector_properties.md)
- [What's New in Revit 2010 API - ImportInstance elements](./0901-33_importinstance_elements.md)
- [What's New in Revit 2010 API - LocationCurve.ElementsAtJoin](./0901-34_locationcurveelementsatjoin.md)
- [What's New in Revit 2010 API - Element material quantities](./0901-35_element_material_quantities.md)
- [What's New in Revit 2010 API - Instance.Transformed and Geometry of Instances](./0901-36_instancetransformed_and_geometry_of_instances.md)
- [What's New in Revit 2010 API - PlaceGroup()](./0901-37_placegroup.md)
- [What's New in Revit 2010 API - gbXMLParamElem.ShadingSurfaces](./0901-38_gbxmlparamelemshadingsurfaces.md)
- [What's New in Revit 2010 API - Creating topography surfaces](./0901-39_creating_topography_surfaces.md)
- [What's New in Revit 2010 API - Changes to Printing](./0901-40_changes_to_printing.md)
- [What's New in Revit 2010 API - Geometry extraction](./0901-41_geometry_extraction.md)
- [What's New in Revit 2010 API - Additional enums](./0901-42_additional_enums.md)
