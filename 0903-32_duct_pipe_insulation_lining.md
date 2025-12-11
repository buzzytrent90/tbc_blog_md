---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:23:48.151754'
original_url: https://thebuildingcoder.typepad.com/blog/0903_whats_new_2012.html
parent_post: 0903_whats_new_2012.md
part_number: '32'
part_total: '45'
post_number: 0903
series: whats_new_in_revit_api
slug: whats_new_2012_duct_&_pipe_insulation_&_lining
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
title: What's New in Revit 2012 API - Duct & pipe insulation & lining
word_count: 2034
---

### Duct & pipe insulation & lining

The new classes

- DuctInsulation- PipeInsulation- DuctLining

and related types support read/write and create access to duct & pipe insulation and lining. In Revit 2012, these objects are now accessible as standalone elements related to their parent duct, pipe, or fitting.

# Small enhancements & API interface changes

## SiteLocation and City TimeZone

The properties

- SiteLocation.Latitude- SiteLocation.Longitude

now use Revit's TimeZone calculation engine to assign an appropriate time zone for the coordinates. (Previously the time zone was not modified when these values were changed).

SiteLocation retains the ability to set the TimeZone manually as the calculation is may not be accurate for locations near the boundaries.

The City class has been adjusted to return the more accurate TimeZone values.

The new static method

- SunAndShadowSettings.CalculateTimeZone()

provides direct access to the results of a time zone calculation.

## FamilyParameter.GUID property

Returns the GUID of a particular family parameter. Allows you to determine if family parameters are shared or not.

## InternalDefintion.Visible property

Identifies if a parameter definition represents a visible or hidden parameter (hidden applies to shared parameters only).

## Selection.GetElementIds() method

Returns a collection containing the ids of the selected elements. This collection can be used directly with FilteredElementCollector.

## API to prompt for rubber band box

- PickBoxStyle- A new enum that controls the style of the pick box.- PickedBox-A new class that contains two XYZ points representing the pick box on the screen.- The following two new methods are added to Selection class that invoke a general purpose two-click editor that lets the user to specify a rectangular area on the screen:

```
PickedBox Selection.PickBox(PickBoxStyle style);
PickedBox Selection.PickBox(PickBoxStyle style, String statusPrompt)
```

## Save and SaveAs APIs permitted in most event handlers

The limitation against calling Save/SaveAs has been removed for most events.

The restriction against calling Save/SaveAs remains only in the following special events:

- DocumentSaving- DocumentSaved- DocumentSavingAs- DocumentSavedAs- DocumentSynchronizingWithCentral- DocumentSynchronizedWithCentral- FileExporting- FileImporting- DocumentPrinting- ViewPrinting- DocumentClosing

Please note that other restrictions may still prevent a successful save (e.g. save cannot be performed if a transaction group or transaction is still opened by the API client or via Revit's UI when the event handler is invoked)

## Opening IFC Documents

A new method allows opening an IFC Document. This method is similar in behaviour to OpenDocumentFile rather than to standard Import methods. It opens the specified file as a newly created document rather than importing it into an existing one. The new document is retuned by the method if opening was successful.

- Application.OpenIFCDocument(String fileName)

## RevitUIFamilyLoadOptions

The class RevitUIFamilyLoadOptions is no longer available for direct construction in the API. If you want to trigger the Revit UI to respond to situations when a loaded family is already found in the target project, obtain a special IFamilyLoadOptions instance from the new static method UIDocument.GetRevitUIFamilyLoadOptions() (in RevitAPIUI.dll) instead.

## Reference documentation for BuiltInParameter members

The members of the BuiltInParameter enum (which are parameter "ids" for Revit use) now have automatically generated documentation. The documentation for each id includes the parameter name, as found in the Element Properties dialog in the English version of Autodesk Revit. Note that multiple distinct parameter ids may map to the same English name; in those case you must examine the parameters associated with a specific element to determine which parameter id to use.

## SolidSolidCutUtils.CanTwoElemsHaveSolidSolidCut() method

This method has been removed and replaced by

- SolidSolidCutUtils.CanElementCutElement()

The new method provides a reason why the cutting element cannot cut the other element.

## Adaptive component API

The methods in the new classes

- AdaptiveComponentFamilyUtils- AdaptiveComponentInstanceUtils

provide access to data related to adaptive component families and instances.

## ViewSheet.ConvertToRealSheet()

This new method converts a placeholder sheet to a real view sheet, optionally applying a titleblock at the same time.

## Initial View Settings

A new class

- IntitialViewSettings

allows access to the initial view settings for a document that controls which view should initially be shown when the model is opened.

It has the following public methods/properties:

- GetInitialViewSettings – Returns the initial view settings for the specified document.- ViewId – Returns (if set) or sets the Id of an initial view- IsAcceptableInitialView – Checks whether the given view is acceptable as an initial view.

## Document preview

The new method

- Document.GetDocumentPreviewSettings()

gets the settings related to the stored document preview image for a given document. It returns a DocumentPreviewSettings object, whose members include:

- DocumentPreviewSettings.PreviewViewId – the id of the view to use for the preview. This value is stored in the document and used in subsequent save operations.- DocumentPreviewSettings.ForceViewUpdate() – sets the document to update the view before saving (useful when the document is never displayed)

Note that it is also possible to temporarily assign a preview view id for one save operation through the SaveOptions and SaveAsOptions classes. The id set to these classes is not stored in the saved documented.

## Document identification

An override for

- Document.Equals()

was added to determine if two Documents represent the same document currently opened in the Revit session.

An override for

- Document.GetHashCode()

was added to return the same hashcode for document instances that represent the same document currently opened in the Revit session.

## Dynamic Update Framework API changes

There are new settings to flag an updater as optional. Optional updaters will not cause prompting the end user when they are opening a document which was modified by that updater but the updater is not currently registered. red). Optional updaters should be used only when necessary. By default, updaters are non-optional. New methods introduced to support this change are:

- UpdaterRegistry.SetIsUpdaterOptional(UpdaterId id, bool isOptional)- UpdaterRegistry.RegisterUpdater(UpdaterId id, bool isOptional)- UpdaterRegistry.RegisterUpdater(UpdaterId id, Document doc, bool isOptional)

New methods were added to access information about currently registered updaters:

- UpdaterRegistry.GetRegisteredUpdaterInfos(Document doc)- UpdaterRegistry.GetRegisteredUpdaterInfos()

Revit now disallows any calls to UpdaterRegistry from within the Execute() method of an updater. That means any calls to RegistryUpdater(), AddTrigger(), etc. will now throw an exception. The only method of UpdaterRegistry allowed to be called during execution of an updater is UnregisterUpdater(,) but the updater to be unregistered must be not the one currently being executed.

## Enclosure class renamed

The Enclosure class, introduced in Revit 2011 as the parent class for Rooms, Spaces and Areas, was renamed to SpatialElement.

## Room, Area and Space boundary segments

The individual BoundarySegment properties for the Room, Space and Area classes have been marked obsolete.

The SpatialElement class contains a new method:

- SpatialElement.GetBoundarySegments()

which works for all subclass types.

## Line origin and direction

Two new properties added to Line class to get the origin and direction of a line:

- Line.Origin- Line.Direction

## TextElement properties

Five new read-only properties have been added to the TextElement class:

- TextElement.UpDirection – The direction towards the top of the text.- TextElement.BaseDirection – The base direction for the text.- TextElement.LineWidth – The TextElement line width.- TextElement.Height – The TextElement height.- TextElement.Align – The TextAlignFlags of the TextElement.

## FaceSplitter class

The FaceSplitter class, representing the element produced by a Split Face operation, has been exposed to the API.

Use this class to identify the element whose face was split by the element (SplitElementId property).
```csharp
  Autodesk.Revit.DB.Options opt
    = app.Create.NewGeometryOptions();
  opt.ComputeReferences = true;
  opt.IncludeNonVisibleObjects = true;

  FilteredElementCollector collector
    = new FilteredElementCollector( doc );

  ICollection<FaceSplitter> splitElements
    = collector.OfClass( typeof( FaceSplitter ) )
      .Cast<FaceSplitter>().ToList();

  foreach( FaceSplitter faceSplitter in
    splitElements )
  {
    Element splitElement = doc.get\_Element(
      faceSplitter.SplitElementId );

    Autodesk.Revit.DB.GeometryElement geomElem
      = faceSplitter.get\_Geometry( opt );

    foreach( GeometryObject geomObj in
      geomElem.Objects )
    {
      Line line = geomObj as Line;
      if( line != null )
      {
        XYZ end1 = line.get\_EndPoint( 0 );
        XYZ end2 = line.get\_EndPoint( 1 );
        double length = line.ApproximateLength;
      }
    }
  }
```

To find the faces created by the split, use the new Face.GetFaceRegions() method on the face of the host element for the split.

## ColumnAttachment

The newly exposed ColumnAttachment class represents an attachment of the top or bottom of a column to a roof, floor, ceiling, or beam. Static methods:

- ColumnAttachment.GetColumnAttachment()- ColumnAttachment.AddColumnAttachment()- ColumnAttachment.RemoveColumnAttachment()

provide access to the settings in this class for a given element.

## Color class

The API color class may represent an invalid or uninitialized color when an instance of it is obtained from Revit. Invalid colors cannot be read; a new exception will throw from the Red, Green, and Blue properties if you attempt to read them. Setting the color properties is permitted and makes the color no longer invalid.

The property

- Color.IsValid

identifies if the color is valid.

## FamilyInstance: flip work plane

The new property

- FamilyInstance.IsWorkPlaneFlipped

is a settable property capable of changing if the work plane for a particular family instance is flipped.

The new property

- FamilyInstance.CanFlipWorkPlane

identifies if the family instance allows flipping of the work plane.

## New NewFamilyInstance() overloads

Two new overloads are provided for creating family instances from References:

- Autodesk.Revit.Creation.ItemFactoryBase.NewFamilyInstance(Reference, Line, FamilySymbol)- Autodesk.Revit.Creation.ItemFactoryBase.NewFamilyInstance(Reference, XYZ, XYZ, FamilySymbol)

These are identical to their counterparts that accept Faces as input. Because the Reference member will not always be populated in all Face handles, these overloads allow an alternate means of creating the target family instance attached to the right face.

The new overload

- Autodesk.Revit.Creation.ItemFactoryBase.NewFamilyInstance(Line, FamilySymbol, View)

provides the ability to create a line-based detail component.

## Allow and disallow wall end joins

The new methods

- WallUtils.DisallowWallJoinAtEnd()- WallUtils.AllowWallJoinAtEnd()- WallUtils.IsWallJoinAllowedAtEnd()

provide access to the setting for whether or not joining is allowed at a particular end of the wall. If joins exist at the end of a wall and joins are disallowed, the walls will become disjoint. If joins are disallowed for the end of the wall, and then the setting is toggled to allow the joins, the wall will automatically join to its neighbors if there are any.

## Element.Pinned property

The Element.Pinned property is no longer read-only. It now allows you to set an element to be pinned or unpinned.

## ElementFilter.PassesElement

Two new overloads allow you to evaluate the result of an ElementFilter:

- bool ElementFilter.PassesElement(Element)- bool ElementFilter.PassesElement(Document, ElementId)

## ElementMulticategoryFilter

This new ElementFilter subtype allows you to easily find elements whose category matches any of a given set of categories.

## ElementMulticlassFilter

This new ElementFilter subtype allows you to easily find elements whose class type matches any of a given set of classes.

## ElementPhaseStatus

The new method

- Element.GetPhaseStatus()

returns the status of a given element in the input phase. Options include none (elements unaffected by phasing), new, demolished, past, future, existing and temporary.

The new element filter allows you to filter for elements which have a status matching one of the set of input statuses.

## Temporary view mode information

The new method:

- View.IsInTemporaryViewMode()

identifies if a particular temporary view mode is active for a view.

The new method:

- View.IsElementVisibleInTemporaryViewMode()

identifies if an element should be visible in the indicated view mode. This applies only to the TemporaryHideIsolate and AnalyticalModel view modes.

## Generalized Array, Set, Map classes removed

The Revit API classes in Autodesk.Revit.Collections:

- Array- Set- Map

have been removed. More flexible alternatives exist in the .NET framework System.Collections and System.Collections.Generic namespaces.

## Replacement for ExternalCommandData.Data

The property

- ExternalCommandData.Data

has been replaced by

- ExternalCommandData.JournalData

The data type is now an IDictionary<String, String>.

The previous Data property has been marked Obsolete.

## Replacement for Application.LibraryPaths

The property

- Application.LibraryPaths

has been replaced by two methods

- Application.GetLibraryPaths()- Application.SetLibraryPaths()

The data type is now an IDictionary<String, String>.

The previous LibraryPaths property has been marked Obsolete.

## LinkElementId property changes

The members of the LinkElementId class have changed in order to clarify what element is represented by an instance of this object. The new properties are:

- LinkElementId.LinkInstanceId – The id of the link, or invalidElementId if no link.- LinkElementId.LinkedElementId – The id of the element in the link, or invalidElementId if no link.- LinkElementId.HostElementId – The id of the element in the host, or invalidElementId if there is a link.

## Application properties

The new properties:

- Application.Username- Application.CentralServerName- Application.LocalServerName

provide read access to information in the current Revit session.

## RevitAddInUtility.dll

This DLL is now compiled as a compatible binary capable of execution on 32-bit or 64-bit systems.

## VSTA changes
---

## Related Sections
- [What's New in Revit 2012 API - Unexpected Places, and Warmer](./0903-01_unexpected_places_and_warmer.md)
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
