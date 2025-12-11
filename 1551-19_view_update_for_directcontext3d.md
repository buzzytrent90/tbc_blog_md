---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:10.438204'
original_url: https://thebuildingcoder.typepad.com/blog/1551_whats_new_2018.html
parent_post: 1551_whats_new_2018.md
part_number: '19'
part_total: '23'
post_number: '1551'
series: geometry
slug: whats_new_2018_view_update_for_directcontext3d
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
title: API Changes - View update for DirectContext3D
word_count: 267
---

### View update for DirectContext3D

The new method:

* UIDocument.UpdateAllOpenViews()

updates all open views in this document after elements have been changed, deleted, selected or de-selected. Graphics in the views are fully redrawn regardless of which elements have changed. This function should only rarely be needed, but might be required when working with graphics drawn from outside of Revit's transactions and elements, for example, when using DirectContext3D.

This function is potentially expensive as many views may be updated at once, including regeneration of view's geometry and redisplay of graphics. Thus for most situations it is recommended that API applications rely on the Revit application framework to update views more deliberately.

## Coordination Model elements

Coordination Model elements current can link the graphical contents of Navisworks files and display them in context in the Revit session. These elements leverage the DirectContext3D framework to handle the display of the external graphics, and are the first example of an element which is designated to contain a link to externally managed DirectContext3D graphics (a "DirectContext3D handle" element).

There is no current way to create new Coordination Model or DirectContext3D handle elements via the API. However, you can use the capabilities of the related classes to identify and manipulate these elements. These elements can be accessed from the following new API classes:

* DirectContext3DHandleUtils – provides utilities related to the identification of types and instances which are storing externalized graphics via DirectContext3D
* DirectContext3DHandleSettings – provides access to override settings applied to DirectContext3D handles through the Visibility/Graphics dialog.
* DirectContext3DHandleOverrides – provides access to DirectContext3DHandleSettings that are stored by a given view.

## Shared Coordinates API additions
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
- [API Changes - RuledSurface API](./1551-18_ruledsurface_api.md)
- [API Changes - Acquire and Publish coordinates API additions](./1551-20_acquire_and_publish_coordinates_api_additions.md)
- [API Changes - SiteLocation API additions](./1551-21_sitelocation_api_additions.md)
- [API Changes - ProjectLocation API additions](./1551-22_projectlocation_api_additions.md)
- [API Changes - Revit Link API additions](./1551-23_revit_link_api_additions.md)
