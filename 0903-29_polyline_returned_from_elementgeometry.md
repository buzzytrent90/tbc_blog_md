---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:23:48.148114'
original_url: https://thebuildingcoder.typepad.com/blog/0903_whats_new_2012.html
parent_post: 0903_whats_new_2012.md
part_number: '29'
part_total: '45'
post_number: 0903
series: whats_new_in_revit_api
slug: whats_new_2012_polyline_returned_from_element.geometry
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
title: What's New in Revit 2012 API - PolyLine returned from Element.Geometry
word_count: 3252
---

### PolyLine returned from Element.Geometry

A new geometry object called a PolyLine is exposed through the API. The PolyLine represents a set of coordinate points forming contiguous line segments. Typically this type of geometry would be seen in geometry imported from other formats (such as DWG). Previous Element.Geometry[] would skip extraction of these geometry object completely.

## Analysis of Room and Space 3D geometry

The new method:

- SpatialElementGeometryCalculator.CalculateSpatialElementGeometry()

computes the 3D geometry of the input spatial element (room or space) and returns it, along with information about the elements which form the boundary of the element.

The new classes:

- SpatialElementGeometryResults- SpatialElementBoundarySubface

encapsulate the results of the geometric calculation.

The class:

- SpatialElementBoundaryOptions

provides the available options for the calculation (currently limited to an option to calculate the boundaries at the finish face or at the boundary object's centerlines and whether to include the free boundary faces in the calculation result).

## Detailed Energy Analysis Model API

A new API is provided to obtain and analyze the contents of a project's detailed energy analysis model, as seen in the Export to gbXML and the Heating and Cooling Loads features:
![Detailed energy analysis model](img/whats_new_2012_1.png)

This analysis produces an analytical thermal model from the physical model of a building. The analytical thermal model is composed of spaces, zones and planar surfaces that represent the actual volumetric elements of the building.

The new classes in Autodesk.Revit.DB.Analysis namespace:

- EnergyAnalysisDetailModel- EnergyAnalysisDetailModelOptions- EnergyAnalysisOpening- EnergyAnalysisSpace- EnergyAnalysisSurface- Polyloop

can be used to generate and analyze the contents of the detailed energy analysis model. Use

- EnergyAnalysisDetailModel.Create()

to create and populate the model (while setting up appropriate options); use

- EnergyAnalysisDetailModel.GetAnalyticalSpace()- EnergyAnalysisDetailModel.GetAnalyticalSurfaces()- EnergyAnalysisDetailModel.GetAnalyticalOpenings()- EnergyAnalysisDetailModel.GetAnalyticalShadingSurfaces()

to extract the entities from the analysis; and use

- EnergyAnalysisDetailModel.Destroy()

to clean up the Revit database after finishing with the analysis results.

## Conceptual energy analysis API

The new classes in the Autodesk.Revit.DB.Analysis namespace:

- ConceptualConstructionType- ConceptualSurfaceType- MassEnergyAnalyticalModel- MassLevelData- MassSurfaceData- MassZone

provide access to the elements and objects created by Revit to perform energy analyses on conceptual design models.

The method

- Document.Export(string,string,MassGBXMLExportOptions)

supports export of a gBXML file containing conceptual energy analysis elements (mass elements) only.

## Analysis visualization framework

The analysis visualization framework was improved to support multiple analysis results shown in the same view at the same time.

The new class

- AnalysisResultSchema

was added to store meta-data for each analysis result. The Results Visibilty view frame control in the user interface and the API's SpatialFieldManager.ResultsEnabledInView and AnalysisResultSchema.IsVisible properties control which results (if any) are displayed.

The class SpatialFieldManager now has new methods:

- RegisterResult()- GetResultSchema()- SetResultSchema()- GetRegisteredResults()

to register and access results meta-data. Corresponding methods and properties of SpatialFieldManager:

- SetUnits()- CurrentUnits- GetDescription()- SetDescription()

are deprecated. They can be used if SpatialFieldManager contains only one analysis result, but they throw an exception if multiple results are registered.

The method

- UpdateSpatialFieldPrimitive()

is changed to take a result index as an additional argument; the old version is deprecated and can be used if SpatialFieldManager contains only one analysis result.

New classes allow for different types of analysis data and different appearance of results:

- AnalysisDisplayDiagramSettings- AnalysisDisplayVectorSettings

```csharp
  // Create a SpatialFieldManager for the active view
  SpatialFieldManager sfm = SpatialFieldManager
    .CreateSpatialFieldManager(doc.ActiveView, 1);
  int primitiveIndex = sfm.AddSpatialFieldPrimitive();

  // This example creates two result schema.
  // Each schema contains a single value at the origin.
  IList<XYZ> pts = new List<XYZ>();
  pts.Add(XYZ.Zero);

  FieldDomainPointsByXYZ pnts = new FieldDomainPointsByXYZ(pts);

  // Create the schema

  AnalysisResultSchema resultSchemaA = new AnalysisResultSchema(
    "Schema A", "Time");

  AnalysisResultSchema resultSchemaB = new AnalysisResultSchema(
    "Schema B", "Distance");

  // Data in Schema A measures time and can be
  // displayed using Hours or Minutes for units

  List<string> unitsList = new List<string>();
  unitsList.Add("Hours");
  unitsList.Add("Minutes");
  List<double> unitsMultipliers = new List<double>();
  unitsMultipliers.Add(1);
  unitsMultipliers.Add(60);
  resultSchemaA.SetUnits(unitsList, unitsMultipliers);

  List<double> doubleList = new List<double>();

  // The data value in Schema A is 3.5 hours

  doubleList.Add(3.5);
  IList<ValueAtPoint> valueList
    = new List<ValueAtPoint>();
  valueList.Add(new ValueAtPoint(doubleList));
  FieldValues fieldValuesA
    = new FieldValues(valueList);

  // Data in Schema B measures distance and can be
  // displayed using Feet or Inches for units
  unitsList.Clear();
  unitsMultipliers.Clear();
  unitsList.Add("Feet");
  unitsList.Add("Inches");
  unitsMultipliers.Add(1);
  unitsMultipliers.Add(12);
  resultSchemaB.SetUnits(unitsList, unitsMultipliers);

  doubleList.Clear();
  valueList.Clear();
  // The data value in Schema B is 5 feet
  doubleList.Add(5);
  valueList.Add(new ValueAtPoint(doubleList));
  FieldValues fieldValuesB = new FieldValues(valueList);

  // Update the view's spatial field primitive with the schema
  sfm.UpdateSpatialFieldPrimitive(primitiveIndex,
    pnts, fieldValuesA, sfm.RegisterResult(resultSchemaA));
  sfm.UpdateSpatialFieldPrimitive(primitiveIndex,
    pnts, fieldValuesB, sfm.RegisterResult(resultSchemaB));
```

## Point Cloud API

Revit offers two sets of APIs related to Point Clouds.

The client API is capable of working with point cloud instances within Revit (creating them, manipulating their properties, and reading the points found matching a given volumetric filter).

The major classes of the client API are:

- PointCloudType – a type of a point cloud, representing the points obtained from a single file or engine's identifier.- PointCloudInstance – an instance of a point cloud in a location in the Revit project.- PointCloudFilter – a filter determining the volume of interest when extracting points.- PointCollection – a collection of points obtained from an instance and a filter.- PointIterator – an iterator for the points in a PointCollection.- CloudPoint – an individual point cloud point, representing an X, Y, Z location in the coordinates of the cloud, and a color.

There are two methods to access the points as a client:

1. In the traditional IEnumerable interface, you can iterate the resulting points directly from the PointCollection.- In an unsafe interface usable only from C# and C++/CLI, you can get a pointer to the point storage of the collection and access the points directly in memory. Although you must deal with pointers directly, there may be performance improvements when traversing large buffers of points.

The following snippets show how to iterate part of a point cloud using both methods. The same point cloud filter is applied for both routines:

#### Reading point cloud points by iteration

```csharp
private int ReadPointCloud\_Iteration(
  PointCloudInstance pcInstance )
{
  PointCloudFilter filter = CreatePointCloudFilter(
    pcInstance.Document.Application, pcInstance );

  // Get points.  Number of points is
  // determined by the needs of the client

  PointCollection points = pcInstance.GetPoints(
    filter, 10000 );

  int numberOfPoints = 0;
  foreach( CloudPoint point in points )
  {
    // Process each point
    System.Drawing.Color color
      = System.Drawing.ColorTranslator.FromWin32(
        point.Color );

    String pointDescription = String.Format(
      "({0}, {1}, {2}, {3}",
      point.X, point.Y, point.Z, color.ToString() );

    numberOfPoints++;
  }
  return numberOfPoints;
}
```

#### Reading point cloud points by pointer

```csharp
public unsafe int ReadPointCloud\_Pointer(
  PointCloudInstance pcInstance )
{
  PointCloudFilter filter = CreatePointCloudFilter(
    pcInstance.Document.Application, pcInstance );

  // Get points.  Number of points is
  // determined by the needs of the client

  PointCollection points = pcInstance.GetPoints(
    filter, 10000 );

  int totalCount = points.Count;
  CloudPoint\* pointBuffer = (CloudPoint\*) points
    .GetPointBufferPointer().ToPointer();

  for( int numberOfPoints = 0;
    numberOfPoints < totalCount; numberOfPoints++ )
  {
    CloudPoint point = \*( pointBuffer + numberOfPoints );

    // Process each point

    System.Drawing.Color color
      = System.Drawing.ColorTranslator.FromWin32(
        point.Color );

    String pointDescription = String.Format(
      "({0}, {1}, {2}, {3}",
      point.X, point.Y, point.Z, color.ToString() );
  }
  return totalCount;
}
```

#### Point cloud filter creation

```csharp
private PointCloudFilter CreatePointCloudFilter(
  Application app, PointCloudInstance pcInstance )
{
  // Filter will match 1/8 of the overall point cloud
  // Use the bounding box (filter coordinates
  // are in the coordinates of the model)

  BoundingBoxXYZ boundingBox
    = pcInstance.get\_BoundingBox( null );

  List<Plane> planes = new List<Plane>();

  XYZ midpoint
    = ( boundingBox.Min + boundingBox.Max ) / 2.0;

  // X boundaries
  planes.Add( app.Create.NewPlane(
    XYZ.BasisX, boundingBox.Min ) );
  planes.Add( app.Create.NewPlane(
    -XYZ.BasisX, midpoint ) );

  // Y boundaries
  planes.Add( app.Create.NewPlane(
    XYZ.BasisY, boundingBox.Min ) );
  planes.Add( app.Create.NewPlane(
    -XYZ.BasisY, midpoint ) );

  // Z boundaries
  planes.Add( app.Create.NewPlane(
    XYZ.BasisZ, boundingBox.Min ) );
  planes.Add( app.Create.NewPlane(
    -XYZ.BasisZ, midpoint ) );

  // Create filter

  PointCloudFilter filter = PointCloudFilterFactory
    .CreateMultiPlaneFilter( planes );

  return filter;
}
```

There are two special API-only tools intended to help your client application interact with the user:

1. The SetSelectionFilter() method and FilterAction property of PointCloudInstance allow you to specify a volumetric filter to be applied to the cloud. The parts of the cloud that pass this filter will be rendered differently in the user interface than the rest of the cloud. If the FilterAction is Highlight, the selected part of the cloud will show in highlight color (blue). If the action is Isolate, only the selected part of the cloud will be visible.- The overloaded method Selection.PickBox() invokes a general purpose two-click editor that lets the user to specify a rectagular area on the screen. While this editor makes no changes in Revit as a result of the selections, you can use the returned box to generate a filter and apply a highlight or isolate action to the point cloud.

This example prompts the user to select a portion of the cloud, and creates a highlight filter for it.

#### Prompt for cloud selection and highlight

```csharp
public void PromptForPointCloudSelection(
  UIDocument uiDoc, PointCloudInstance pcInstance )
{
  Application app = uiDoc.Application.Application;
  Selection currentSel = uiDoc.Selection;

  PickedBox pickedBox = currentSel.PickBox(
    PickBoxStyle.Enclosing,
    "Select region of cloud for highlighting" );

  XYZ min = pickedBox.Min;
  XYZ max = pickedBox.Max;

  //Transform points into filter
  View view = uiDoc.ActiveView;
  XYZ right = view.RightDirection;
  XYZ up = view.UpDirection;

  List<Plane> planes = new List<Plane>();

  // X boundaries
  bool directionCorrect = IsPointAbovePlane(
    right, min, max );
  planes.Add( app.Create.NewPlane( right,
    directionCorrect ? min : max ) );
  planes.Add( app.Create.NewPlane( -right,
    directionCorrect ? max : min ) );

  // Y boundaries
  directionCorrect = IsPointAbovePlane(
    up, min, max );
  planes.Add( app.Create.NewPlane( up,
    directionCorrect ? min : max ) );
  planes.Add( app.Create.NewPlane( -up,
    directionCorrect ? max : min ) );

  // Create filter
  PointCloudFilter filter = PointCloudFilterFactory
    .CreateMultiPlaneFilter( planes );

  Transaction t = new Transaction(
    uiDoc.Document, "Highlight" );
  t.Start();

  pcInstance.SetSelectionFilter( filter );
  pcInstance.FilterAction
    = SelectionFilterAction.Highlight;

  t.Commit();
}

private static bool IsPointAbovePlane(
  XYZ normal, XYZ planePoint, XYZ point )
{
  XYZ difference = point - planePoint;
  difference = difference.Normalize();
  double dotProduct = difference.DotProduct( normal );
  return dotProduct > 0;
}
```

The engine API is capable of supplying points in a point cloud to Revit. A custom engine implementation consists of the following:

- An implementation of IPointCloudEngine registered to Revit via the PointCloudEngineRegistry.- An implementation of IPointCloudAccess coded to respond to inquiries from Revit regarding the properties of a single point cloud.- An implementation of IPointSetIterator code to return sets of points to Revit when requested.

Engine implementations may be file-based or non-file-based:

- File-based implementations require that each point cloud be mapped to a single file on disk. Revit will allow users to create new point cloud instances in a document directly by selecting point cloud files whose extension matches the engine identifier. These files are treated as external links in Revit and may be reloaded and remapped when necessary from the Manage Links dialog.- Non-file-based engine implementations may obtain point clouds from anywhere (e.g. from a database, from a server, or from one part of a larger aggregate file). Because there is no file that the user may select, Revit's user interface will not allow a user to create a point cloud of this type. The engine provider should supply a custom command using PointCloudType.Create() and PointCloudInstance.Create() to create and place point clouds of this type. The Manage Links dialog will show the point clouds of this type, but since there is no file associated to the point cloud, the user cannot manage, reload or remap point clouds of this type.

Regardless of the type of engine used, the implementation must supply enough information to Revit to display the contents of the point cloud. There are two ReadPoints methods which must be implemented:

- IPointCloudAccess.ReadPoints() – this provides a single set of points in a one-time call from Revit. Revit uses this during some display activities including selection prehighlighting. It is also possible for API clients to call this method directly (via PointCloudInstance.GetPoints()).- IPointSetIterator.ReadPoints() – this provides a subset of points as a part of a larger iteration of points in the cloud. Revit uses this method during normal display of the point cloud; quantities of points will be requested repeatedly until it obtains enough points or until something in the display changes. The engine implementation must keep track of which points have been returned to Revit during any given point set iteration.

## Material API changes and PropertySets

The Revit Materials API is largely renovated to allow for a representation of materials that is both more compact and more extensible.

The following classes are now obsolete:

- MaterialWood- MaterialConcrete- MaterialSteel- MaterialGeneric- MaterialOther

The properties on these classes are now accessible from PropertySets or the Material class itself.

All named properties on Material specific to appearance or structure (e.g. Shininess) are also obsolete.

In their place, a material will have one or more aspects pertaining to rendering appearance, structure, or other major material category. Each aspect is represented by a PropertySet or PropertySetElement. Each material can own its properties of an aspect via a PropertySet or share them with other materials as a reference to a PropertySetElement.

New enumerated types:

- MaterialAspect – An enumeration of material aspects. Currently, we support rendering and structural aspects.

New classes:

- PropertySetElement – an Element containing a PropertySet – used for sharing PropertySets among materials.

New methods:

- Material.Create()- PropertySetElement.Create()- Material.SetMaterialAspectByPropertySet()- Material.SetMaterialAspectIndependent()- Material.GetMaterialAspectPropertySet()

## Performance Adviser

The new Revit feature Performance adviser is designed to analyze the document and flag for the user any elements and/or settings that may cause performance degradation. The Performance Adviser command executes set of rules and displays their result in a standard review warnings dialog.

The API for performance adviser consists of 2 classes (PerformanceAdviser and IPerformanceAdviserRule). PerformanceAdviser is an application-wide singleton that has a dual role: it is a registry of performance checking rules and an engine to execute them. The methods of PerformanceAdviser:

- AddRule()- DeleteRule()- GetNumberOfRules()- GetRuleIDs()- GetRuleName()- GetRuleDescription()- ExecuteRules()- ExecuteAllRules()

allow you to manipulate what rules are checked. Applications that create new rules are expected to use AddRule() to register the new rule during application startup and DeleteRule() to deregister it during application shutdown.

Methods of PerformanceAdviser

- SetRuleEnabled- IsRuleEnabled- ExecuteRules

allow UI or API application to mark rules for execution and run them on a given document, getting report as a list of FailureMessage objects.

The new interface IPerformanceAdviserRule allows you to define new rules for the Performance Adviser. Your application should create a class implementing this interface, instantiate an object of the derived class and register it using PerformanceAdviser.AddRule(). Methods of IPerformanceAdviserRule available for override include:

- GetName()- GetDescription()

which provide rule identification information;

- InitCheck()- FinalizeCheck()

which are executed once per check and can be used to perform checks of the document "as a whole";

- WillCheckElements()- GetElementFilter()- ExecuteElementCheck()

which allow the rule to identify a subset of elements in the document to be checked and run the check on the individual elements.

Potentially problematic results found during rule execution are reported by returning FailureMessage(s).

## External File References (Linked Files)

The API can now tell what elements in Revit are references to external files, and can make some modifications to where Revit loads external files from.

An Element which contains an ExternalFileReference is an element which refers to some external file (ie. a file other than the main .rvt file of the project.) Two new Element methods, IsExternalFileReference() and GetExternalFileReference(), let you get the ExternalFileReference for a given Element.

ExternalFileReference contains methods for getting the path of the external file, the type of the external file, and whether the file was loaded, unloaded, not found, etc. the last time the main .rvt file was opened.

The classes RevitLinkType and CADLinkType can have IsExternalFileReference() return true. RevitLinkTypes refer to Revit files linked into another Revit project. CADLinkTypes refer to DWG files. Note that CADLinkTypes can also refer to DWG imports, which are not external file references, as imports are brought completely into Revit. A property IsImport exists to let users distinguish between these two types.

Additionally, the element which contains the location of the keynote file is an external file reference, although it has not been exposed as a separate class.

There is also a class ExternalFileUtils, which provides a method for getting all Elements in a document which are references to external files.

Additionally, the classes ModelPath and ModelPathUtils have been exposed. ModelPaths can store paths to a location on disk, a network drive, or a Revit Server location. ModelPathUtils provides methods for converting between modelPath and String.

Finally, the class TransmissionData allows users to examine the external file data in a closed Revit document, and to modify path and load-state information for that data. Two methods, ReadTransmissionData, and WriteTransmissionData, are provided. With WriteTransmissionData, users can change the path from which to load a given link, and can change links from loaded to unloaded and vice versa. (WriteTransmissionData cannot be used to add links to or remove links from a document, however.)

Newly exposed classes:

- ExternalFileReference – A non-Element class which contains path and type information for a single external file which a Revit project references. ExternalFileReference also contains information about whether the external file was loaded or unloaded the last time the associated Revit project was opened.- ExternalFileUtils – A utility class which allows the user to find all external file references, get the external file reference from an element, or tell whether an element is an external file reference.- RevitLinkType – An element representing a Revit file linked into a Revit project.- CADLinkType – An element representing a DWG drawing. CADLinkTypes may be links, which maintain a relationship with the file they originally came from, or imports, which do not maintain a relationship. The property IsImport will distinguish between the two kinds.- LinkType – The base class of RevitLinkType and CADLinkType.- ModelPath – A non-Element class which contains path information for a file (not necessarily a .rvt file.) Paths can be to a location on a local or network drive, or to a Revit Server location.- ModelPathUtils – A utility class which provides methods for converting between strings and ModelPaths.- TransmissionData – A class which stores information about all of the external file references in a document. The TransmissionData for a Revit project can be read without opening the document.

## Customizing IFC export

The classes

- ExporterIFCRegistry- IExporterIFC

allow your custom application to override the default implementation for the IFC export process.

The interface is passed an ExporterIFC object. The ExporterIFC object is the starting point for all IFC export activities and offers access to the options selected for the given export operation. It also contains access to information cached by Revit during the export process; this information is cached to provide easy access to it later, and sometimes to write it to the file at the end of the export process.

There are several auxiliary objects provided to support implementation of an IFC export client. These are the most important:

- IFCFile – a representation of the IFC file being written. This class contains methods which directly generate handles to IFC entries and write them to the file.- IFCAnyHandle – a wrapper around any sort of IFC element, product, or other data.- IFCLabel – a string in an IFC file.- IFCMeasureValue – a parameterized value in an IFC file.- ExporterIFCUtils – a collection of utilities to support the Revit IFC export client.

The IFC export client for Revit 2012 represents a transitional state between the version implemented internally in Revit 2011 and the final state which should be written 100% using the Revit API. Temporary APIs are exposed to bridge the gaps between the API client code and portions of the previous implementation. As Autodesk continues to evolve this export client, temporary APIs will be changed, deprecated and/or removed in future versions of Revit.

## MEP API major enhancements
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
