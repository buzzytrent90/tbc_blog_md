---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:10.429058'
original_url: https://thebuildingcoder.typepad.com/blog/1551_whats_new_2018.html
parent_post: 1551_whats_new_2018.md
part_number: '11'
part_total: '23'
post_number: '1551'
series: geometry
slug: whats_new_2018_api_additions
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
title: API Changes - API Additions
word_count: 526
---

### API Additions

## API to get list of reviewable warnings from a Document

The new method:

* Document.GetWarnings()

returns a list of failure messages generated from persistent (reviewable) warnings accumulated in the document.

## API access to FamilyInstance references

The following new methods have been added to enable easy access to FamilyInstance references that correspond to reference planes and reference lines in the family. Some use the options in the new enumeration FamilyInstanceReferenceType as input to identify "Strong" or "Weak" references or specific positional references in each of the 3 coordinate directions (as determined by the possible values of parameter "Is Reference" of reference planes and parameter "Reference" of reference lines in families).

* FamilyInstance.GetReferences()
* FamilyInstance.GetReferenceByName()
* FamilyInstance.GetReferenceType()
* FamilyInstance.GetReferenceName()

## Multistory Stairs API

The new class:

* MultistoryStairs

allows users to create stairs that span multiple levels. A multistory stairs element may contain multiple stairs whose extents are governed by base and top levels.

This element will contain one or more Stairs elements. Stairs elements are either a reference instance which is copied to each level covered by groups of identical stairs instances which share the same level height, or individual Stairs instances which are not connected to a group with the same level height. By default, when adding new levels to the multistory stair, new stairs will be added to the group.

For groups of duplicate stairs at different levels, the instances can be found as Subelements of the Stairs element.

Stairs in a connected group can be edited together by modifying the associated Stairs instance. For specific floors that need special designs, stairs can be separated from a group by unpinning the element, changes made to this Stairs will not affect other any other instance in the element, or add the stairs back into the group if needed. However, any changes made to the stair will be lost since the stair's properties will be overridden by the group specifications.

The class has the following methods:

* MultistoryStairs.AddStairsByLevelIds() – Adds stairs to the given levels.
* MultistoryStairs.RemoveStairsByLevelIds() – Removes stairs from the given levels. This will regenerate the multistory stairs from the remaining levels.
* MultistoryStairs.CanAddStair() – Checks if the input level id can be used to add stairs into multistory stairs.
* MultistoryStairs.CanRemoveStair() – Checks if the input level id can be used to remove stairs from the multistory stairs.
* MultistoryStairs.GetAllConnectedLevels() – Gets the ids of all levels connected to the multistory stairs.
* MultistoryStairs.GetAllStairsIds() – Gets the ids of all of the stairs in the multistory stairs.
* MultistoryStairs.Create() – Creates a multistory stairs object.
* MultistoryStairs.GetStairsConnectedBaseLevelIds() – Gets the base level ids for the stairs contained in this multi-story stairs element.
* MultistoryStairs.IsPinned() – Checks if a stair is pinned as a propagation group.
* MultistoryStairs.Unpin() – Unpins a story of stairs by giving its base level id.
* MultistoryStairs.Pin() – Pins a unpinned stairs back into a story of a stairs.

The new property:

* Stairs.MultistoryStairsId

indicates the id of the MultistoryStairs element to which the Stairs belong to.

Related to StairsPath functionality for multistory stairs, the new functions:

* StairsPath.CanCreateOnMultistoryStairs()
* StairsPath.CreateOnMultistoryStairs()

support creation of new stairs paths in a plan view for stairs instances in a multistory stairs element.
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
- [API Changes - Railings API additions related to MultistoryStairs](./1551-12_railings_api_additions_related_to_multistorystairs.md)
- [API Changes - DimensionEqualityLabelFormating API](./1551-13_dimensionequalitylabelformating_api.md)
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
