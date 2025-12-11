---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:23:48.135866'
original_url: https://thebuildingcoder.typepad.com/blog/0903_whats_new_2012.html
parent_post: 0903_whats_new_2012.md
part_number: '15'
part_total: '45'
post_number: 0903
series: whats_new_in_revit_api
slug: whats_new_2012_boolean_operations
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
title: What's New in Revit 2012 API - Boolean operations
word_count: 523
---

### Boolean operations

The new methods

- BooleanOperationsUtils.ExecuteBooleanOperation()- BooleanOperationsUtils.ExecuteBooleanOperationModifyingOriginalSolid()

execute a boolean operation combining a pair of solid geometry objects. Options to the method include the operations type: Union, Difference, or Intersect.

The first method takes a copy of the input solids and produces a new solid as a result. Its first argument can be any solid, either obtained directly from a

Revit element or created via another operation like GeometryCreationUtils.

The second method performs the boolean operation directly on the first input solid. The first input must be a solid which is not obtained directly from a Revit

element. The property

- GeometryObject.IsElementGeometry

can identify whether the solid is appropriate as input for this method.

In this example, the geometry of intersecting columns and walls is obtained by a Boolean intersection operation. The intersection volume and number of boundary faces is shown in the resulting dialog.
```csharp
/// <summary>
/// A data structure containing the details
/// of each intersecting wall/column pair.
/// </summary>
struct Intersection
{
  public Element Wall;
  public Element Column;
  public Solid Solid;
}

/// <summary>
/// A collection of all intersections.
/// </summary>
private List<Intersection> m\_allIntersections;

/// <summary>
/// Finds and posts information on wall/column intersections.
/// </summary>
/// <param name="doc">The document.</param>
public void FindIntersectionVolumes( Document doc )
{
  // Find all Wall elements.
  FilteredElementCollector collector
    = new FilteredElementCollector( doc );
  collector.OfClass( typeof( Wall ) );
  m\_allIntersections = new List<Intersection>();
  foreach( Wall wall in collector.OfType<Wall>() )
  {
    // Find all intersecting columns
    FilteredElementCollector columnIntersectionCollector
      = new FilteredElementCollector( doc );
    // Columns may be one of two different categories
    List<BuiltInCategory> categories
      = new List<BuiltInCategory>();
    categories.Add( BuiltInCategory.OST\_Columns );
    categories.Add( BuiltInCategory.OST\_StructuralColumns );
    ElementMulticategoryFilter categoryFilter
      = new ElementMulticategoryFilter( categories );
    // Apply intersection filter to find matches
    ElementIntersectsElementFilter intersectsFilter
      = new ElementIntersectsElementFilter( wall );
    columnIntersectionCollector
      .WhereElementIsNotElementType()
      .WherePasses( categoryFilter )
      .WherePasses( intersectsFilter );
    foreach( Element element in
      columnIntersectionCollector )
    {
      // Store information on intersection
      Intersection intersection;
      intersection.Wall = wall;
      intersection.Column = element;
      Solid wallSolid = GetGeometry( wall );
      Solid columnSolid = GetGeometry( element );
      // Intersect the solid geometry of the two elements
      intersection.Solid = BooleanOperationsUtils
        .ExecuteBooleanOperation( wallSolid,
        columnSolid, BooleanOperationsType.Intersect );
      m\_allIntersections.Add( intersection );
    }
  }
  TaskDialog td = new TaskDialog( "Intersection info" );
  td.MainInstruction = "Intersections found: "
    + m\_allIntersections.Count;
  StringBuilder builder = new StringBuilder();
  foreach( Intersection intersection in
    m\_allIntersections )
  {
    builder.AppendLine( String.Format(
      "{0} x {1}: volume {2} faces {3}",
      intersection.Wall.Name,
      intersection.Column.Name,
      intersection.Solid.Volume,
      intersection.Solid.Faces.Size ) );
  }
  td.MainContent = builder.ToString();
  td.Show();
}

/// <summary>
///  Gets the solid geometry of an element.
/// </summary>
/// <remarks>Makes an assumption that each element
/// consists of only one postive-volume solid, and
/// returns the first one it finds.</remarks>
/// <param name="e"></param>
/// <returns></returns>
private Solid GetGeometry( Element e )
{
  GeometryElement geomElem = e.get\_Geometry(
    new Options() );
  foreach( GeometryObject geomObj in
    geomElem.Objects )
  {
    // Walls and some columns will have a
    // solid directly in its geometry
    if( geomObj is Solid )
    {
      Solid solid = (Solid)geomObj;
      if( solid.Volume > 0 )
        return solid;
    }
    // Some columns will have a instance
    // pointing to symbol geometry
    if( geomObj is GeometryInstance )
    {
      GeometryInstance geomInst
        = (GeometryInstance) geomObj;
      // Instance geometry is obtained so that the
      // intersection works as expected without
      // requiring transformation
      GeometryElement instElem
        = geomInst.GetInstanceGeometry();
      foreach( GeometryObject instObj in
        instElem.Objects )
      {
        if( instObj is Solid )
        {
          Solid solid = (Solid)instObj;
          if( solid.Volume > 0 )
            return solid;
        }
      }
    }
  }
  return null;
}
```
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
