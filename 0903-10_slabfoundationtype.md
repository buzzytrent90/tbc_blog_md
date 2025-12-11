---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:23:48.131387'
original_url: https://thebuildingcoder.typepad.com/blog/0903_whats_new_2012.html
parent_post: 0903_whats_new_2012.md
part_number: '10'
part_total: '45'
post_number: 0903
series: whats_new_in_revit_api
slug: whats_new_2012_slabfoundationtype
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
title: What's New in Revit 2012 API - SlabFoundationType
word_count: 1656
---

### SlabFoundationType

The enumerated value SlabFoundationType.Slab is now SlabFoundationType.SlabOneWay. A new option, SlabTwoWay, is also available. Existing floors assigned the value Slab will report SlabOneWay.

# Major enhancements to the Revit API

## Extensible Storage

The Revit API now allows you to create your own class-like Schema data structures and attach instances of them to any Element in a Revit model. This functionality can be used to replace the technique of storing data in hidden shared parameters. Schema-based data is saved with the Revit model and allows for higher-level, metadata-enhanced, object-oriented data structures. Schema data can be configured to be readable and/or writable to all users, just a specific application vendor, or just a specific application from a vendor.

The extensible storage classes are all found in Autodesk.Revit.DB.ExtensibleStorage

- Autodesk.Revit.DB.ExtensibleStorage.SchemaBuilder -- Used to create Schemas definitions- Autodesk.Revit.DB.ExtensibleStorage.Schema --Contains a unique schema identifier, read/write permissions, and a collection of data Field objects.- Autodesk.Revit.DB.ExtensibleStorage.Field -- Contains data name, type, and unit information and is used as the key to access corresponding data in an Element- Autodesk.Revit.DB.ExtensibleStorage.FieldBuilder -- A helper class used with SchemaBuilder used when creating a new field.- Autodesk.Revit.DB.ExtensibleStorage.Entity -- An object containing data corresponding to a Schema that can then be inserted into an Element.

The following data types are currently supported:

- int- short- double- float- bool- string- Guid- ElementId- Autodesk.Revit.DB.XYZ- Autodesk.Revit.DB.UV- Autodesk.Revit.DB.ExtensibleStorage.Entity (An instance of another Schema, also known as a SubSchema)- Array (as a System.Collections.Generic.IList<T>)- Map (as a System.Collections.Generic.IDictionary<TKey, TValue>)- All types supported for simple types, including Entity, are supported for the <TValue> parameter.- All types supported for simple types \*except\* double, float, XYZ, UV, and Entity are supported for the <TKey> parameter.

#### Simple usage of ExtensibleStorage

```csharp
// Create a data structure, attach it to a wall,
// populate it with data, and retrieve the data
// back from the wall
public void StoreDataInWall(
  Wall wall,
  XYZ dataToStore )
{
  Transaction createSchemaAndStoreData
    = new Transaction(wall.Document, "tCreateAndStore");
  createSchemaAndStoreData.Start();
  SchemaBuilder schemaBuilder = new SchemaBuilder(
    new Guid("720080CB-DA99-40DC-9415-E53F280AA1F0"));
  // allow anyone to read the object
  schemaBuilder.SetReadAccessLevel(AccessLevel.Public);
  // restrict writing to this vendor only
  schemaBuilder.SetWriteAccessLevel(AccessLevel.Vendor);
  // required because of restricted write-access
  schemaBuilder.SetVendorId("ADSK");
  // create a field to store an XYZ
  FieldBuilder fieldBuilder = schemaBuilder
    .AddSimpleField("WireSpliceLocation", typeof(XYZ));
  fieldBuilder.SetUnitType(UnitType.UT\_Length);
  fieldBuilder.SetDocumentation(
    "A stored location value representing a wiring splice in a wall.");
  schemaBuilder.SetSchemaName("WireSpliceLocation");
  // register the Schema object
  Schema schema = schemaBuilder.Finish();
  // create an entity (object) for this schema (class)
  Entity entity = new Entity(schema);
  // get the field from the schema
  Field fieldSpliceLocation
    = schema.GetField("WireSpliceLocation");
  // set the value for this entity
  entity.Set<XYZ>(fieldSpliceLocation,
    dataToStore, DisplayUnitType.DUT\_METERS);
  // store the entity in the element
  wall.SetEntity(entity);
  // get the data back from the wall
  Entity retrievedEntity = wall.GetEntity(schema);
  XYZ retrievedData = retrievedEntity.Get<XYZ>(
    schema.GetField("WireSpliceLocation"),
    DisplayUnitType.DUT\_METERS);
  createSchemaAndStoreData.Commit();
}
```

## Worksharing API

Several new classes were added to provide access to worksharing information in the document:

- Workset – Represents a workset in the document. Worksets are a way to divide a set of elements in the Revit document into subsets for worksharing.- WorksetId – Identifies a workset within a single document.- WorksetKind – An enumerated type that indicates one of the standard kinds of workset (as available in the UI).- WorksetTable – A table containing references to all the worksets contained in a document.- WorksetVisibility – An enumerated type that indicates the visibility settings of a workset in a particular view.- WorksetDefaultVisibilitySettings – An object that manages default visibility of worksets in a document.- FilteredWorksetCollector – This class is used to search, filter and iterate through a set of worksets. Developers can assign a condition to filter the worksets that are returned. If no condition is applied, it attempts to access all the worksets in the document.- FilteredWorksetIdIterator – An iterator to a set of workset ids filtered by the settings of a FilteredWorksetCollector.- FilteredWorksetIterator – An iterator to a set of worksets filtered by the settings of a FilteredWorksetCollector.- WorksetFilter – A base class for a type of filter that accepts or rejects worksets based upon criteria.- WorksetKindFilter – A filter used to match worksets of the given WorksetKind.- ElementWorksetFilter – A filter used to match elements which reside in a given workset (use this filter with FilteredElementCollector).- WorksharingUtils – access to information about a work-shared document.- WorksharingTooltipInfo – basic read-only information about a work-shared document, such as owner, creator, etc.

Some related additions were made to existing classes:

- Document.GetWorksetTable() – Gets the WorksetTable of this document. There is one WorksetTable for each document.- Document.GetWorksetId(ElementId id) – Gets the id of the Workset which owns the element.- Element.WorksetId – Gets the id of the workset which owns the element.- View.GetWorksetVisibility(WorksetId worksetId) – Returns the visibility settings of a workset for this particular view.- View.SetWorksetVisibility(WorksetId worksetId, WorksetVisibility visible) – Sets visibility for a workset in this view. This setting overrules implicit visibility of the workset for this particular view.- View.IsWorksetVisible(WorksetId worksetId) – Indicates whether the workset is visible in this view.

In addition, there is API support for the new 2012 worksharing visualization functionality:

- View.SetWorksharingDisplayMode and View.GetWorksharingDisplayMode allow the API to control which worksharing display mode is enabled in the view.- WorksharingDisplaySettings allows getting and setting the specific graphic overrides that will be applied in the various worksharing display modes.

## Setting the Active View

The new property

- UIDocument.ActiveView

has both a getter and setter, so it allows you to query the currently active view of the currently active document, and also allows you to set it similarly to what an end user can do by changing a view in the Project Browser in Revit.

The setter has a number of limitations:

- It can only be used in an active document, which must not be in read-only state and must not be inside a transaction.- The setter also cannot be used during ViewActivating and ViewActivated events, or during any pre-action event, such as DocumentSaving, DocumentExporting, or other similar events.

## Opening and activating a document

A new method:

- UIApplication.OpenAndActivateDocument(String)

was added to the UI API. It opens a Revit document and makes it the active one. The document is opened with its default view displayed.

There are limitations preventing this method to be called at certain situations:

- when there is a transaction open in the currently active document (if there is an active document)- during execution of any event handler

## Adding a custom Ribbon tab

The new methods

- UIApplication.CreateRibbonTab()- UIApplication.CreateRibbonPanel(string, string)- UIApplication.GetRibbonPanels(string)

(and the corresponding methods in UIControlledApplication) provide the ability to add a new ribbon tab to Revit, at the end of the list of static tabs (to the right of the Add-Ins tab, if shown). If multiple tabs are added, they will be shown in the order added.

There is a limit to the number of custom tabs supported in a given session of Revit (20). This limit is provided to ensure that the standard tabs remain visible and usable. Because of this, your application should only add a custom tab if it's really needed.

## Construction modeling API

New functionality in Revit 2012 allows elements to be divided into sub-parts, collected into assemblies, and displayed in special assembly views. The API for dividing parts is still under development and likely to change.

Read, write and create access to assemblies in the Revit environment is provided through the classes:

- Autodesk.Revit.DB.Assembly.AssemblyInstance- Autodesk.Revit.DB.Assembly.AssemblyType

A new assembly containing the selected elements can be created as follows:
```csharp
  ElementId categoryId = doc.get\_Element(
    uidoc.Selection.GetElementIds().
  FirstOrDefault() ).Category.Id;

  ElementId titleblockId
    = doc.TitleBlocks.Cast<FamilySymbol>()
      .First<FamilySymbol>().Id;

  AssemblyInstance instance = null;

  Transaction t = new Transaction( doc );

  if( AssemblyInstance.IsValidNamingCategory( doc,
    categoryId, uidoc.Selection.GetElementIds() ) )
  {
    t.SetName( "Create Assembly Instance" );
    t.Start();
    instance = AssemblyInstance.Create( doc,
      uidoc.Selection.GetElementIds(), categoryId );
    t.Commit();

    t.SetName( "Set Assembly Name" );
    t.Start();
    string assemblyName = "Assembly #1";
    if( AssemblyInstance.IsValidAssemblyName( doc,
      assemblyName, categoryId ) )
    {
      instance.AssemblyTypeName = assemblyName;
    }
    t.Commit();
  }
```

Other important methods include AssemblyInstance.GetMemberIds(), AssemblyInstance.SetMemberIds(), and AssemblyInstance.Disassemble().

Assembly views, that display only the elements in the assembly, are created with the AssemblyViewUtils class such as:
```csharp
  if( instance.AllowsAssemblyViewCreation() )
  {
    ViewSheet viewSheet = AssemblyViewUtils
      .CreateSheet( doc, instance.Id, titleblockId );

    View3D view3d = AssemblyViewUtils
      .Create3DOrthographic( doc, instance.Id );

    ViewSection detailSectionA = AssemblyViewUtils
      .CreateDetailSection( doc, instance.Id,
      AssemblyDetailViewOrientation.DetailSectionA );

    View materialTakeoff = AssemblyViewUtils
      .CreateMaterialTakeoff( doc, instance.Id );

    View partList = AssemblyViewUtils
      .CreatePartList( doc, instance.Id );
  }
```

The PartUtils class provides methods to identify Part elements that are created by sub-dividing model elements. These methods describe the relationship between Parts and the elements (such as walls, floors, etc) that are divided to create the Parts.

## DB-level applications

The add-in framework has been extended to support database-level add-ins. These add-ins should be used when the purpose of your application is to assign events and/or updaters to the Revit session, but not to add anything to the Revit user interface or use APIs from RevitAPIUI.dll.

To implement a DB-level application, implement the methods in the Autodesk.Revit.DB.IExternalDBApplication interface:

- public Result OnStartup(Autodesk.Revit.ApplicationServices.ControlledApplication app)- public Result OnShutdown(Autodesk.Revit.ApplicationServices.ControlledApplication app)

Within the OnStartup() method you should register events and updaters which your application will respond to during the session.

To register the DB-level application with Revit, add the appropriate registry entry to a manifest file in the Addins folder.
A DB-level application has a similar structure as for a UI external application:
```csharp
<?xml version="1.0" standalone="no"?>
<RevitAddIns>
  <AddIn Type="DBApplication">
    <Assembly>MyDBLevelApplication.dll</Assembly>
    <AddInId>DA3D570A-1AB3-4a4b-B09F-8C15DFEC6BF0</AddInId>
    <FullClassName>MyCompany.MyDBLevelAddIn</FullClassName>
    <Name>My DB-Level AddIn</Name>
  </AddIn>
</RevitAddIns>
```

## Geometry API enhancements
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
