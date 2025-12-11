---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:10.421686'
original_url: https://thebuildingcoder.typepad.com/blog/1551_whats_new_2018.html
parent_post: 1551_whats_new_2018.md
part_number: '05'
part_total: '23'
post_number: '1551'
series: geometry
slug: whats_new_2018_external_commands_and_applications
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
title: API Changes - External commands and applications
word_count: 528
---

### External commands and applications

External API commands and applications are now enabled by default in perspective views. The following behavior applies:

* Any external command that was automatically disabled by Revit when a perspective view is active will now be active. No code changes are necessary to make this happen.
* Any external command that was explicitly disabled in perspective views will remain disabled. For example, if your command has an accessibility function limiting it to use in certain view types, the command will not be accessible in perspective views unless that function accepts ViewType.ThreeD as a view type that is allowed.
* Macros, the Macro Manager tools, Dynamo scripts and the Dynamo editor are also newly enabled when a perspective view is active.

## Subelements

Several Revit elements can now contain a subdivision known as a Subelement. Subelements provide a way for parts of an element to behave as though they were real elements without incurring the overhead of adding more full elements to the model.

Many Revit features – for example parameters, schedules, and tags – were designed to operate on Elements. As a result, the Revit code needs to represent objects as Elements for them to participate in those features. This can lead to scalability problems, because every Element adds overhead and adding many Elements may decrease the performance of the model.

An alternative is to use Subelements. An element can expose a set of "Subelements" that it contains, specifying characteristics like their category and parameters, and certain Revit capabilities will treat those Subelements the same as ordinary Elements. For example, a Subelement may contribute geometry to the main element and may be able to be selected independently of its parent Element. It will possibly have its own (settable) type as well as an assigned category which can be different from its parent Element.

In the API, the new Subelement class is used to refer to either an Element or a specific subelement of a given Element. It is typically directly related to a Reference to either the Element or the specific subelement.

Note that creation of new Subelements for a given element is not done generically. Instead, the given Element may provide the ability to modify its definition, resulting in the creation of new Subelements.

Examples of Elements which may incorporate Subelements in practice include:

* Rebar
* RebarContainer
* FabricSheet
* Stairs elements which make up MultistoryStairs elements
* Railing
* ContinuousRail

To get access to a particular Subelement, you may use any of the following:

* Subelement.Create()
* Subelement.IsValidSubelementReference()
* Document.GetSubelement(Reference) – Gets the subelement referenced by the input reference.
* Document.GetSubelement(String uniqueId) – Gets the subelement referenced by a unique id string.
* Element.GetSubelements() – Returns the collection of the Element's Subelements.

To access the basic Subelement properties, including its category and geometry, use:

* Subelement.GetBoundingBox()
* Subelement.GetGeometryObject()
* Subelement.IsModifiable()
* Subelement.Document
* Subelement.Element
* Subelement.Category
* Subelement.GetReference()
* Subelement.UniqueId
* ExportUtils.GetExportId(Subelement)

To access the Subelement's type, use:

* Subelement.TypeId
* Subelement.ChangeTypeId()
* Subelement.GetValidTypes()
* Subelement.IsValidType()
* Subelement.CanHaveTypeAssigned()

To access the Subelement's parameters, use:

* Subelement.GetAllParameters()
* Subelement.GetParameterValue()
* Subelement.SetParameterValue()
* Subelement.IsParameterModifiable()
* Subelement.HasParameter()

For Elements which allow deletion of individual Subelements, use:

* Element.DeleteSubelement()
* Element.DeleteSubelements()
* Element.CanDeleteSubelement()
---

## Related Sections
- [API Changes - What's New in the Revit 2018 API](./1551-01_whats_new_in_the_revit_2018_api.md)
- [API Changes - Table of Contents](./1551-02_table_of_contents.md)
- [API Changes - API Changes](./1551-03_api_changes.md)
- [API Changes - Element modifications and contextual commands](./1551-04_element_modifications_and_contextual_commands.md)
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
- [API Changes - View update for DirectContext3D](./1551-19_view_update_for_directcontext3d.md)
- [API Changes - Acquire and Publish coordinates API additions](./1551-20_acquire_and_publish_coordinates_api_additions.md)
- [API Changes - SiteLocation API additions](./1551-21_sitelocation_api_additions.md)
- [API Changes - ProjectLocation API additions](./1551-22_projectlocation_api_additions.md)
- [API Changes - Revit Link API additions](./1551-23_revit_link_api_additions.md)
