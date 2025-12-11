---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:23:48.122772'
original_url: https://thebuildingcoder.typepad.com/blog/0903_whats_new_2012.html
parent_post: 0903_whats_new_2012.md
part_number: '01'
part_total: '45'
post_number: 0903
series: whats_new_in_revit_api
slug: whats_new_2012_unexpected_places,_and_warmer
source_file: 0903_whats_new_2012.htm
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
- rooms
- schedules
- selection
- sheets
- transactions
- views
- walls
- windows
title: What's New in Revit 2012 API - Unexpected Places, and Warmer
word_count: 2369
---

﻿

# What's New in the Revit 2012 API

This is the third instalment of a series publishing the information provided in the 'What's New' sections of the past few Revit API releases help file RevitAPI.chm.

The first instalment covering
[What's New in the Revit 2010 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2010-api.html) explains
my motivation for this and provides an overview of the other releases.

We now move on to the Revit 2012 API, looking at:

- [Major renovations](#2)
- [Major enhancements](#3)
- [Small enhancements and changes](#4)

First, however, another update on my vacation.

### Unexpected Places, and Warmer

I am still on holiday in Italy, so please do not expect any immediate responses to comments for a while.

I love adventures, and finding myself in unexpected places, and I am getting my fill of that here and now.

I found better weather further south, and a beautiful empty beach north of Bari.

Acting on a recommendation by a cyclist whom I asked about his unintelligible dialect in a bar, I also ended up enjoying the unique
[trulli](http://en.wikipedia.org/wiki/Trullo) of
[Alberobello](http://en.wikipedia.org/wiki/Alberobello).

![Trulli in Alberobello](img/Trulli_Alberobello.jpg)

From there we continued to
[Lecce](http://en.wikipedia.org/wiki/Lecce) and
started exploring the sweet and efficient little local train system connecting many of the communities on the peninsula of
[Salento](http://en.wikipedia.org/wiki/Salento).

We visited Galatina, continued to Nardo, and walked a long way on foot towards Santa Caterina to reach the protected natural park and beach of
[Porto Selvaggio](http://it.wikipedia.org/wiki/Parco_naturale_regionale_Porto_Selvaggio_e_Palude_del_Capitano).

![Portoselvaggio](img/Portoselvaggio.jpg)

In spite of some rain and cold, with some warm sunshine in between, we spent a day or two in pure uninterrupted nature.

On the way out towards Nardo again, a passing car picked us up and took us to Galipoli, another surprise visit, with a direct train connection back to Lecce.

Now, to continue the promised series of 'What's New' documentation from the past few Revit API releases.

# Major renovations to the Revit 2012 API

## .NET 4.0 now used by the Revit API

The Revit API has been enhanced to run with the .NET 4.0 runtime. As a result, Visual Studio 2010 with a framework target of .NET 3.5 or 4.0 must be used to debug your addins. Addins compiled with Visual Studio 2008 will run normally outside the debugging environment.

All samples in the Revit API SDK have been upgraded to Visual Studio 2010.

Microsoft has not yet announced when Visual Studio Tools for Applications (VSTA) for .NET 4.0 will be available. VSTA is the technology used for Revit macros. The Microsoft VSTA debugger is not compatible with the default Revit 2012 .NET 4.0 environment. VSTA macros can run successfully in the .NET 4.0 environment, but a special Revit configuration is required for debugging. In order to use VSTA debugging, you must:

1. Ensure you have user permissions sufficient to modify files in the Revit installation folder.- Exit Revit.- Run '[Revit.exe directory]\RevitVSTAConfig.exe'- Click the 'Prepare Revit for VSTA' button.- Restart Revit.

Note: Revit's native components continue to be compiled with the VS 2008 C++ compiler. Therefore Revit 2012 does not include the VS2010 redistributable. Third-party applications which include natively compiled C++ components should use the VS 2008 C++ compiler or must include the VS 2010 C++ redistributables with the installation.

## RegenerationMode

Automatic regeneration mode has been removed. You no longer have to include the RegenerationMode attribute on your add-ins. All APIs expect that add-in code is written using manual regeneration when necessary.

## Add-In registration – new required property VendorId

Two new registration properties have been added:

- VendorId – a mandatory string conforming to the Autodesk vendor ID standard. Register your vendor id string with Autodesk at http://www.autodesk.com/symbreg. An error will be shown if the add-in manifest file does not contain this required node for add-in entry.- VendorDescription – optional string containing vendor's legal name and/or other pertaining information.

Here is an example of the new entries:
```csharp
<?xml version="1.0" encoding="utf-16" standalone="no"?
<RevitAddIns>
  <AddIn Type="Command">
    <Assembly>Command.dll</Assembly>
    <ClientId>d7e30025-97d4-4012-a581-5f8ed8d18808</ClientId>
    <FullClassName>Revit.Command</FullClassName>
    <Text>Command</Text>
    <VendorId>ADSK</VendorId>
    <VendorDescription>Autodesk, www.autodesk.com</VendorDescription>
  </AddIn>
</RevitAddIns>
```

## CompoundStructure and WallSweeps

The CompoundStructure class has been replaced. The old CompoundStructure class in the API was read-only and supported only the nominal layers list without support for the settings related to vertical regions, sweeps, and reveals. The new CompoundStructure class supports read and modification of all of the structure information include vertical regions, sweeps and reveals.

The CompoundStructure.Layers property has been replaced by CompoundStructure.GetLayers() and CompoundStructure.SetLayers().

The CompoundStructureLayer class has also been replaced for the same reasons as CompoundStructure.

The following table maps properties from the old CompoundStructureLayer to the new version of the class:

- DeckProfile – DeckProfileId: Is now an ElementId.- DeckUsage – DeckEmbeddingType: Uses a different enum.- Function – Function: Uses a different enum.- Material – MaterialId: Is now an ElementId.- Thickness – Width.- Variable – N/A: Not a part of the layer class, use CompoundStructure.VariableLayerIndex instead.

The property HostObjAttributes.CompoundStructure has been replaced by two methods:

- HostObjAttributes.GetCompoundStructure()- HostObjAttributes.SetCompoundStructure()

Remember that you must set the CompoundStructure back to the HostObjAttributes instance in order for any change to be stored in the element.

In addition to the information on wall sweeps found in the CompoundStructure class, there is a new API representing a wall sweep or reveal. The WallSweep class is an element that represents either a standalone wall sweep/reveal, or one added by the settings in a given wall's compound structure. Standalone wall sweeps and reveals may be constructed using the static method Create().

## LinePattern

The LinePattern class has been replaced. The old LinePattern class in the API represented both the line pattern itself and the element that contains it, and offered no details on the contents of the pattern beyond its name. The new classes available are:

- LinePatternElement – an element that contains a line pattern- LinePattern – the line pattern. This class provides access to the pattern name and the list of segments that make up the pattern. The line segments define a repeating pattern of dashes and dots for the line pattern.

The method

- LinePatternElement.Create()

offers the ability to add new LinePattern elements to the Revit database.

The property of the Settings class:

- Settings.LinePatterns

has been removed. LinePatterns may be found by the following approaches:

1. Use of a FilteredElementCollector filtering on class LinePatternElement- Use of the static methods of LinePatternElement:
     - LinePatternElement.GetLinePattern(Document, ElementId)- LinePatternElement.GetLinePatternByName(Document, string)

## FillPattern

The FillPattern class has been replaced. The old FillPattern class in the API represented both the fill pattern itself and the element that contains it, and offered no details on the contents of the pattern beyond its name. The new classes available are:

- FillPatternElement – an element that contains a fill pattern- FillPattern – the fill pattern. This class provides access to the pattern name and the set of grids that make up the pattern.- FillGrid – a single fill pattern grid, described in the two dimensions of a face.

The method

- FillPatternElement.Create()

offers the ability to add new FillPattern elements to the Revit database.

The property of the Settings class:

- Settings.FillPatterns

has been removed. FillPatterns may be found by the following approaches:

1. Use of a FilteredElementCollector filtering on class FillPatternElement- Use of the static methods of FillPatternElement:
     - FillPatternElement.GetFillPattern(Document, ElementId)- FillPatternElement.GetFillPatternByName (Document, string)

## IndependentTag

A good portion of the IndependentTag class and related classes have been renovated.

- Leader has been renamed HasLeader.- LeaderMode has been renamed LeaderEndCondition, and the members of the LeaderEndCondition enum have been renamed.- A new method CanLeaderEndConditionBeAssigned() determines if the LeaderEndCondition can be set.- TagMode has been replaced by multiple properties: IsMaterialTag, IsMulticategoryTag.- Members of the TagOrientation enum have been renamed.

Some new members were added to determine the elements referenced by the tag:

- New property TaggedElementId – provides the id of the element referenced by the tag.- New property TaggedLocalElementId – provides the id of a linked element reference by the tag.- New method GetTaggedLocalElement() – returns the handle of the element reference by the tag.- New property IsOrphaned – Orphans are those tags that are associated with an instance of a linked Revit file but have no host element. Tags become orphaned when the element they were tagging was deleted from the link.

## Import and Export APIs changes

The import and export APIs no longer have special argument combinations which permit use of the "active view".
If you wish to export from or import to the active view, you must obtain it directly and pass it as input to the method.

- For the following APIs, a non-empty ViewSet must be provided:
  - bool Export (string folder, string name, ViewSet views, DGNExportOptions options);- bool Export (string folder, string name, ViewSet views, DWGExportOptions options);- bool Export (string folder, string name, ViewSet views, DXFExportOptions options);- bool Export (string folder, string name, ViewSet views, SATExportOptions options);- bool Export (string folder, string name, ViewSet views, DWFExportOptions options);- bool Export (string folder, string name, ViewSet views, DWFXExportOptions options);- bool Export (string folder, string name, ViewSet views, FBXExportOptions options);

- For the following API, a 3D view must be provided:
  - bool Export(string folder, string name, View3D view, ViewPlan grossAreaPlan, BuildingSiteExportOptions options);

- For import APIs, the 'View' property has been removed from the DWGImportOptions and ImageImportOptions. Instead, a new required argument has been added for the following APIs:
  - bool Import (string file, DWGImportOptions options, View view, out Element element);- bool Import (string file, ImageImportOptions options, View view, out Element element);- bool Link (string file, DWGImportOptions options, View view, out Element element);

The methods

- ACADExportOptions.GetPredefinedSetupNames()- DWGExportOptions.GetPredefinedOptions()- DXFExportOptions.GetPredefinedOptions()

provide access to predefined setups and settings from a given document for DWG and DFX export.

The method

- Document.Import(String, InventorImportOptions)

has been replaced by

- Application.OpenBuildingComponentDocument(String)

## Save and Close API changes

- Document.Close()- Document.Close(bool saveModified)

The behaviour of these methods has been changed. Previously, they would prompt the interactive user to pick a target path name if the document's path name was not already set, or if the target file was read-only. Now they will throw an exception if the document's path name is not yet set, or if the saving target file is read-only.

- Document.Save()

The behaviour of this method has been changed. Previously, it would prompt the interactive user to pick a target path name if the document's path name was not already set, or if the target file was read-only. Now it will throw an exception if the document's path name is not set yet. In this case, it needs to be first saved using the SaveAs method instead, or when the target file is read-only.

- Document.Save(SaveOptions)

This new method behaves identically to Document.Save(), but allows you to specify a temporary id to use to generate the file preview.

- Document.SaveAs(string fileName)- Document.SaveAs(string fileName, bool changeDocumentFilename)

The behaviour of these methods has been changed. Previously, they would prompt the user before overwriting another file. Now an exception will be thrown in that situation.

- Document.SaveAs(string fileName, SaveAsOptions options)

This new method allows you to save a document, and encapsulates the options to rename the document in session, overwrite an existing file (if it exists), and temporarily assign a view id to use to generate the file preview.

- UIDocument.SaveAndClose()

This new method closes the document after saving it. If the document's path name has not been set the "Save As" dialog will be shown to the Revit user to set its name and location.

- UIDocument.SaveAs()

This new method saves the document to a file name and path obtained from the Revit user (via the "Save As" dialog).

## Reference properties

The Reference class is being renovated to be more closely aligned with the native Revit class it wraps.
Because of this change, References will no longer carry information on the document, element, or geometry object it was obtained from.
The following table lists the replacement API calls for the obsolete Reference properties:

- Element – ElementId or Document.GetElement(Reference)- GeometryObject – Element.GetGeometryObjectFromReference(Reference)- Transform – ReferenceWithContext.GetInstanceTransform() (see below)- ProximityParameter – ReferenceWithContext.ProximityParameter (see below)- ElementReferenceType – Unchanged- GlobalPoint – Unchanged- UVPoint – Unchanged

The method

- Document.FindReferencesByDirection()

is affected most by this change. The method was the only method to populate the Transform and ProximityParameter properties of Reference. You should switch to the replacement method:

- Document.FindReferencesWithContextByDirection()

This replacement method returns a list of ReferenceWithContext objects, containing the Reference plus additional members:

- ReferenceWithContext.ProximityParameter- ReferenceWithContext.GetInstanceTransform()

Note that methods which existed in Revit 2011 and earlier will continue in this release to return a fully populated Reference object (it is not necessary to change code dealing with these methods). However, methods added in Revit 2012 may not return a fully populated Reference handle and will need to be parsed using the replacement methods above. Also, affected members have been marked obsolete and planned to be removed in a future release.

## Event argument changes

A few changes have been made to event argument classes as we align frameworks to generate code for events.

Note: some of these classes and methods may change again in future.

The following table of changes lists the affected property, the classes providing it, and the new member with optional notes:

- Cancel – Many event argument classes – Methods Cancel(), IsCancelled() – Now available only in argument classes for pre-events that derive from RevitAPIPreEventArgs.- ImportedInstance – FileImportedEventArgs – Property ImportedInstanceId – Now an ElementId.- PrintedViews – DocumentPrintedEventArgs – Method GetPrintedViewElementIds().- FailedViews – DocumentPrintedEventArgs – Method GetFailedViewElementIds().- Views – DocumentPrintingEventArgs – Method GetViewElementIds().

## Event sender changed for UI events

For some UI events on UIApplication and UIControlledApplication, the type of the "sender" object has been changed.

- For the ApplicationClosing event, the sender object is now a UIControlledApplication object (it was previously a ControlledApplication);- For all other UI events, the sender object is now a UIApplication (it was previously an Application).

## Move/Mirror/Rotate/Array changes
---

## Related Sections
- [What's New in Revit 2012 API - Move method replacements](./0903-02_move_method_replacements.md)
- [What's New in Revit 2012 API - Mirror method replacements](./0903-03_mirror_method_replacements.md)
- [What's New in Revit 2012 API - Rotate method replacements](./0903-04_rotate_method_replacements.md)
- [What's New in Revit 2012 API - New Copy methods](./0903-05_new_copy_methods.md)
- [What's New in Revit 2012 API - LinearArray creation replacement methods](./0903-06_lineararray_creation_replacement_methods.md)
- [What's New in Revit 2012 API - RadialArray creation replacement methods](./0903-07_radialarray_creation_replacement_methods.md)
- [What's New in Revit 2012 API - AnalyticalModel now an Element](./0903-08_analyticalmodel_now_an_element.md)
- [What's New in Revit 2012 API - AnalyticalModelSelector](./0903-09_analyticalmodelselector.md)
- [What's New in Revit 2012 API - SlabFoundationType](./0903-10_slabfoundationtype.md)
- [What's New in Revit 2012 API - Get original geometry of a FamilyInstance](./0903-11_get_original_geometry_of_a_familyinstance.md)
- [What's New in Revit 2012 API - Extrusion analysis of a solid](./0903-12_extrusion_analysis_of_a_solid.md)
- [What's New in Revit 2012 API - GeometryCreationUtilities](./0903-13_geometrycreationutilities.md)
- [What's New in Revit 2012 API - Find 3D elements by intersection](./0903-14_find_3d_elements_by_intersection.md)
- [What's New in Revit 2012 API - Boolean operations](./0903-15_boolean_operations.md)
- [What's New in Revit 2012 API - HostObject – top, bottom, side faces](./0903-16_hostobject_top_bottom_side_faces.md)
- [What's New in Revit 2012 API - Get host face of a FamilyInstance](./0903-17_get_host_face_of_a_familyinstance.md)
- [What's New in Revit 2012 API - Element.Geometry](./0903-18_elementgeometry.md)
- [What's New in Revit 2012 API - GeometryObject.GraphicsStyleId](./0903-19_geometryobjectgraphicsstyleid.md)
- [What's New in Revit 2012 API - Curve representation of an Edge](./0903-20_curve_representation_of_an_edge.md)
- [What's New in Revit 2012 API - Centroid of a Solid](./0903-21_centroid_of_a_solid.md)
- [What's New in Revit 2012 API - Transforming geometry](./0903-22_transforming_geometry.md)
- [What's New in Revit 2012 API - Instance.GetTransform() and Instance.GetTotalTransform()](./0903-23_instancegettransform_and_instancegettotaltransform.md)
- [What's New in Revit 2012 API - Serialization/deserialization of References](./0903-24_serializationdeserialization_of_references.md)
- [What's New in Revit 2012 API - Face.HasRegions & Face.GetRegions()](./0903-25_facehasregions_facegetregions.md)
- [What's New in Revit 2012 API - Face.MaterialElementId replaces Face.MaterialElement](./0903-26_facematerialelementid_replaces_facematerialelement.md)
- [What's New in Revit 2012 API - NewHermiteSpline tangency control](./0903-27_newhermitespline_tangency_control.md)
- [What's New in Revit 2012 API - New NewNurbSpline overload](./0903-28_new_newnurbspline_overload.md)
- [What's New in Revit 2012 API - PolyLine returned from Element.Geometry](./0903-29_polyline_returned_from_elementgeometry.md)
- [What's New in Revit 2012 API - Pipe settings and sizes](./0903-30_pipe_settings_and_sizes.md)
- [What's New in Revit 2012 API - Placeholder ducts and pipes](./0903-31_placeholder_ducts_and_pipes.md)
- [What's New in Revit 2012 API - Duct & pipe insulation & lining](./0903-32_duct_pipe_insulation_lining.md)
- [What's New in Revit 2012 API - VSTA enabled for multiple Revit sessions](./0903-33_vsta_enabled_for_multiple_revit_sessions.md)
- [What's New in Revit 2012 API - Track Changes UI](./0903-34_track_changes_ui.md)
- [What's New in Revit 2012 API - LineLoad.UniformLoad](./0903-35_lineloaduniformload.md)
- [What's New in Revit 2012 API - NewBeamSystem() changes](./0903-36_newbeamsystem_changes.md)
- [What's New in Revit 2012 API - NewTruss() change](./0903-37_newtruss_change.md)
- [What's New in Revit 2012 API - Rebar changes](./0903-38_rebar_changes.md)
- [What's New in Revit 2012 API - Spare and space circuits](./0903-39_spare_and_space_circuits.md)
- [What's New in Revit 2012 API - Cable tray and conduit domain](./0903-40_cable_tray_and_conduit_domain.md)
- [What's New in Revit 2012 API - Connector](./0903-41_connector.md)
- [What's New in Revit 2012 API - MEPSystem](./0903-42_mepsystem.md)
- [What's New in Revit 2012 API - Graphical warnings for disconnects](./0903-43_graphical_warnings_for_disconnects.md)
- [What's New in Revit 2012 API - Space properties](./0903-44_space_properties.md)
- [What's New in Revit 2012 API - Fitting methods](./0903-45_fitting_methods.md)
