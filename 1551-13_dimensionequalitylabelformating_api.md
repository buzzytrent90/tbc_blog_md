---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:10.431518'
original_url: https://thebuildingcoder.typepad.com/blog/1551_whats_new_2018.html
parent_post: 1551_whats_new_2018.md
part_number: '13'
part_total: '23'
post_number: '1551'
series: geometry
slug: whats_new_2018_dimensionequalitylabelformating_api
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
title: API Changes - DimensionEqualityLabelFormating API
word_count: 51
---

### DimensionEqualityLabelFormating API

The new class:

* DimensionEqualityLabelFormatting

allows users to set dimension equality formulas for continuous linear or angular dimensions.

New methods and properties include:

* DimensionEqualityLabelFormatting.LeadingSpaces
* DimensionEqualityLabelFormatting.LabelType
* DimensionEqualityLabelFormatting.Prefix
* DimensionEqualityLabelFormatting.Suffix
* DimensionEqualityLabelFormatting.GetFormatOptions()
* DimensionEqualityLabelFormatting.SetFormatOptions()

The following new methods in DimensionType allow access to equality formulas:

* DimensionType.GetEqualityFormula()
* DimensionType.SetEqualityFormula()
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
- [API Changes - UnitsFormatOptions in DimensionType](./1551-14_unitsformatoptions_in_dimensiontype.md)
- [API Changes - OrdinateDimensionSetting](./1551-15_ordinatedimensionsetting.md)
- [API Changes - Surface and Face API](./1551-16_surface_and_face_api.md)
- [API Changes - RevolvedSurface API](./1551-17_revolvedsurface_api.md)
- [API Changes - RuledSurface API](./1551-18_ruledsurface_api.md)
- [API Changes - View update for DirectContext3D](./1551-19_view_update_for_directcontext3d.md)
- [API Changes - Acquire and Publish coordinates API additions](./1551-20_acquire_and_publish_coordinates_api_additions.md)
- [API Changes - SiteLocation API additions](./1551-21_sitelocation_api_additions.md)
- [API Changes - ProjectLocation API additions](./1551-22_projectlocation_api_additions.md)
- [API Changes - Revit Link API additions](./1551-23_revit_link_api_additions.md)
