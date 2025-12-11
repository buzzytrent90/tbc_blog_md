---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:10.436840'
original_url: https://thebuildingcoder.typepad.com/blog/1551_whats_new_2018.html
parent_post: 1551_whats_new_2018.md
part_number: '18'
part_total: '23'
post_number: '1551'
series: geometry
slug: whats_new_2018_ruledsurface_api
source_file: 1551_whats_new_2018.md
tags:
- doors
- elements
- family
- geometry
- levels
- parameters
- python
- references
- revit-api
- rooms
- schedules
- selection
- sheets
- transactions
- views
- walls
title: API Changes - RuledSurface API
word_count: 531
---

### RuledSurface API

The newly added methods:

* RuledSurface.HasFirstProfilePoint()
* RuledSurface.HasSecondProfilePoint()

check if a point was used to define one of the surface profiles.

## Level API addition

The new method:

* Level.FindAssociatedPlanViewId()

finds the id of the first available associated floor or structural plan view associated with this level. If there are multiple associated views, Revit will return the first one it finds.

## Dockable Frame API Additions

Custom Dockable Panes now support the ability for display of dynamic UI elements, such as web browser controls. This capability should be used in cases where the UI for the pane (layout, buttons etc.) changes dynamically during the lifetime of the Revit session. To use this, implement the new interface:

* IFrameworkElementCreator

with a method:

* IFrameworkElementCreator.CreateFrameworkElement()

that constructs and returns the WPF Framework element to embedded in the Revit dockable pane.

The new members:

* DockablePaneProviderData.GetFrameworkElement()
* DockablePaneProviderData.FrameworkElementCreator

provide the ability for the application to deliver a dynamic framework element to the dockable pane.

The property:

* DockablePaneProviderData.FrameworkElement

is now allowed to be null, in situations where the FrameworkElement will be dynamically created.

## DirectContext3D for display of externally managed 3D graphics in Revit

DirectContext3D is an API for displaying external graphics in the context of a Revit model. The API provides a more connected experience to users who can benefit from the ability to display graphics based on geometry that is either difficult or costly to fully import into Revit.

An external plugin can use DirectContext3D API to render geometry by encoding it inside pairs of vertex and index buffers. The communication between Revit and the plugin is accomplished with the use of the External Service Framework (ESF). Revit's rendering pipeline asks registered servers of the DirectContext3D service to provide the geometry for rendering. Revit informs the plugin about certain rendering state, such as the display style and whether the current rendering pass is for transparent objects. The plugin also communicates certain information to Revit, such as the bounding box of the geometry to be rendered.

The following list contains the major added classes and their descriptions:

* DirectContext3D.IDirectContext3DServer – The interface to be implemented by a server of the DirectContext3D external service.
* DirectContext3D.DrawContext – A class that provides drawing functionality for use by DirectContext3D servers.
* DirectContext3D.Vertex – The base class for DirectContext3D vertices.
* DirectContext3D.VertexStream – The base class for DirectContext3D vertex streams, which are used to write vertex data into buffers.
* DirectContext3D.VertexBuffer – A buffer that stores vertex data for rendering.
* DirectContext3D.VertexFormat – A specification of the format of vertex data contained in a piece of geometry.
* DirectContext3D.VertexFormatBits – Vertex format (i.e., the type of data associated with a vertex) represented as a number.
* DirectContext3D.IndexPrimitive – The base class for index buffer primitives.
* DirectContext3D.IndexStream – The base class for DirectContext3D index streams, which are used to write vertex indices into buffers.
* DirectContext3D.IndexBuffer – A buffer that stores vertex indices for rendering.
* DirectContext3D.EffectInstance – An effect instance that controls the appearance of geometry.
* DirectContext3D.PrimitiveType – Type of geometry primitive represented as a number.
* DirectContext3D.ClipPlane – A set of parameters representing a clip plane in DirectContext3D.
* DirectContext3D.ProjectionMethod – The projection method used by a DirectContext3D camera.
* DirectContext3D.Camera – A collection of camera settings for DirectContext3D.
---

## Related Sections
- [API Changes - What's New in the Revit 2018 API](./1551-01_whats_new_in_the_revit_2018_api.md)
- [API Changes - Table of Contents](./1551-02_table_of_contents.md)
- [API Changes - API Changes](./1551-03_api_changes.md)
- [API Changes - Element modifications and contextual commands](./1551-04_element_modifications_and_contextual_commands.md)
- [API Changes - External commands and applications](./1551-05_external_commands_and_applications.md)
- [API Changes - References and selection of subelements](./1551-06_references_and_selection_of_subelements.md)
- [API Changes - Classes](./1551-07_classes.md)
- [API Changes - Methods](./1551-08_methods.md)
- [API Changes - Properties](./1551-09_properties.md)
- [API Changes - Enumerated types](./1551-10_enumerated_types.md)
- [API Changes - API Additions](./1551-11_api_additions.md)
- [API Changes - Railings API additions related to MultistoryStairs](./1551-12_railings_api_additions_related_to_multistorystairs.md)
- [API Changes - DimensionEqualityLabelFormating API](./1551-13_dimensionequalitylabelformating_api.md)
- [API Changes - UnitsFormatOptions in DimensionType](./1551-14_unitsformatoptions_in_dimensiontype.md)
- [API Changes - OrdinateDimensionSetting](./1551-15_ordinatedimensionsetting.md)
- [API Changes - Surface and Face API](./1551-16_surface_and_face_api.md)
- [API Changes - RevolvedSurface API](./1551-17_revolvedsurface_api.md)
- [API Changes - View update for DirectContext3D](./1551-19_view_update_for_directcontext3d.md)
- [API Changes - Acquire and Publish coordinates API additions](./1551-20_acquire_and_publish_coordinates_api_additions.md)
- [API Changes - SiteLocation API additions](./1551-21_sitelocation_api_additions.md)
- [API Changes - ProjectLocation API additions](./1551-22_projectlocation_api_additions.md)
- [API Changes - Revit Link API additions](./1551-23_revit_link_api_additions.md)
