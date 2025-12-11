---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:06.639025'
original_url: https://thebuildingcoder.typepad.com/blog/0902_whats_new_2011.html
parent_post: 0902_whats_new_2011.md
part_number: '06'
part_total: '52'
post_number: 0902
series: whats_new_in_revit_api
slug: whats_new_2011_new_transaction_interfaces
source_file: 0902_whats_new_2011.htm
tags:
- csharp
- elements
- family
- filtering
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
- vbnet
- views
- walls
- windows
title: What's New in Revit 2011 API - New transaction interfaces
word_count: 948
---

### New transaction interfaces

New interfaces have been added to the API related to Transactions. These interfaces will provide a more comprehensive set of options for managing changes being made to the Revit document.

At this time, the new transaction interfaces are not compatible with the automatically-opened transactions created around ExternalCommand callbacks and VSTA macros. So, Autodesk recommends that you not use these interfaces until compatible changes have been made to those frameworks. In order to experiment with the new interfaces, a document needs to be opened or created while in an external command. In the new document, either the old transaction interfaces or the new ones can be used, as long as they are not used together in the same document.

There are three main classes among the new interfaces plus a few supporting classes. The main classes represent three different (but related) transaction contexts.

**Transaction** – a context required in order to make any changes to a Revit model. Only one transaction can be open at a time; nesting is not allowed. Each transaction must have a name, which will be listed on the Undo menu in Revit once a transaction is successfully submitted.

**SubTransaction** – can be used to enclose a set of model modifying commands. Sub-transactions are optional. They are not required in order to modify the model. They are a convenience tool to allow logical splitting of larger tasks into smaller ones. Sub-transaction can only be created within an already opened transaction and must be closed (either committed or rolled back) before the transaction is closed (committed or rolled back). Unlike transactions, sub-transaction may be nested, but any nested sub-transaction must be closed before the enclosing sub-transaction is closed. Sub-transactions do not have name, for they do not appear on the Undo menu in Revit.

**Transaction group** – allows grouping together several independent transactions, which gives the owner of a group an opportunity to treat many transactions at once. When a transaction group is to be closed, it can be rolled back, which means that all already submitted transactions will be rolled back at once. If not rolled back, a group can be either submitted or assimilated. In the former case, all submitted transactions (within the group) will be left as they were. In the later case, transactions within the group will be merged together into one single transaction that will bear the group's name. A transaction group can only be started when there is no transaction open yet, and must be closed only after all enclosed transactions are closed (rolled back or committed). Transaction groups can be nested, but any nested group must be closed before the enclosing group is closed. Transaction groups are optional. They are not required in order to make modifications to a model.

All three transaction objects share some common methods:

- Start – Will start the context.- Commit – Ends the context and commits all changes to the document.- RollBack – Ends the context and discards all changes to the document.- GetStatus – Returns the current status of a transaction object

Besides having the GetStatus returning the current status, both Commit and RollBack methods also return a status indicating whether or not the method was successful. Available status values include:

- Uninitialized – The initial value after object is instantiated; the context has not started yet- Started – Set after a transaction object successfully started (Start method was called)- Committed – Set after a transaction object successfully committed (Commit method was called)- RolledBack – Set after a transaction object successfully rolled back (RollBack method was called)- Pending – Set after transaction object was attempted to be either submitted or rolled back, but due to failures that process could not be finished yet and is waiting for the end-user's response (in a modeless dialog). Once the failure processing is finished, the status will be automatically updated (to either Committed or RolledBack status).

The new Transaction interfaces replace the old APIs:

- Document.BeginTransaction()- Document.EndTransaction()- Document.AbortTransaction()

which have been removed from the API.

#### Refactored Document properties related to transactions

Two existing properties have been re-implemented to better complement the changes in the transaction framework:

- Document.IsReadOny

Indicates that a document is – either temporarily or permanently – in read-only state. If it is read-only, it would mean that new transactions may not be started in that document, thus no modification could be made to the model.

- Document.IsModifiabe

Indicates whether a document is or is not currently modifiable. A document is not modifiable if either there is no transaction currently open or when some temporary state lock modification for a certain operation. Taken from another perspective, a transaction may be started only when document is not yet modifiable, but is also not read-only.

#### Transactions in events

All events have been changed to no longer automatically open transactions like they originally did in earlier releases. As a result of this new policy, the document will not be modified during an event unless one of the event’s handlers modifies it by making changes inside a transaction. A transaction may be opened by either the event handler if there is no transaction already opened (sometimes when the event is invoked, there is already a trasnaction open). If an event handler opens a transaction it is required that it will also close it (commit it or roll it back), otherwise all eventual changes will be discarded.

Please be aware that modifying the active document is not permitted during some events (e.g. the DocumentClosing event). If an event handler attempts to make modifications during such an event, an exception will be thrown. The event documentation indicates whether or not the event is read-only.
---

## Related Sections
- [What's New in Revit 2011 API - Changes to the Revit API namespaces](./0902-02_changes_to_the_revit_api_namespaces.md)
- [What's New in Revit 2011 API - Split of Revit API DLL](./0902-03_split_of_revit_api_dll.md)
- [What's New in Revit 2011 API - New classes for XYZ, UV, and ElementId](./0902-04_new_classes_for_xyz_uv_and_elementid.md)
- [What's New in Revit 2011 API - Replacement for Symbol and properties that access types](./0902-05_replacement_for_symbol_and_properties_that_access_.md)
- [What's New in Revit 2011 API - New ExternalCommand and ExternalApplication registration mechanism](./0902-07_new_externalcommand_and_externalapplication_regist.md)
- [What's New in Revit 2011 API - External Command and External Application registration utility](./0902-08_external_command_and_external_application_registra.md)
- [What's New in Revit 2011 API - Attributes for configuring ExternalCommand and ExternalApplication behavior](./0902-09_attributes_for_configuring_externalcommand_and_ext.md)
- [What's New in Revit 2011 API - New element iteration interfaces](./0902-10_new_element_iteration_interfaces.md)
- [What's New in Revit 2011 API - Replacement for View.Elements](./0902-11_replacement_for_viewelements.md)
- [What's New in Revit 2011 API - Replacement for 3 structural enums](./0902-12_replacement_for_3_structural_enums.md)
- [What's New in Revit 2011 API - Replacement for AnalyticalModel](./0902-13_replacement_for_analyticalmodel.md)
- [What's New in Revit 2011 API - Replacement for gbXMLParamElem](./0902-14_replacement_for_gbxmlparamelem.md)
- [What's New in Revit 2011 API - Revit exceptions](./0902-15_revit_exceptions.md)
- [What's New in Revit 2011 API - Removed deprecated events](./0902-16_removed_deprecated_events.md)
- [What's New in Revit 2011 API - Changes to VSTA](./0902-17_changes_to_vsta.md)
- [What's New in Revit 2011 API - Dynamic Model Update](./0902-18_dynamic_model_update.md)
- [What's New in Revit 2011 API - Elements changed event](./0902-19_elements_changed_event.md)
- [What's New in Revit 2011 API - Failure API](./0902-20_failure_api.md)
- [What's New in Revit 2011 API - Select element(s), point(s) on element(s), face(s) or edge(s)](./0902-21_select_elements_points_on_elements_faces_or_edges.md)
- [What's New in Revit 2011 API - Pick point on the view active workplane](./0902-22_pick_point_on_the_view_active_workplane.md)
- [What's New in Revit 2011 API - Additional options for Ribbon customization](./0902-23_additional_options_for_ribbon_customization.md)
- [What's New in Revit 2011 API - Create and display Revit-style Task Dialogs](./0902-24_create_and_display_revit_style_task_dialogs.md)
- [What's New in Revit 2011 API - Idling event](./0902-25_idling_event.md)
- [What's New in Revit 2011 API - Sun and shadows settings element](./0902-26_sun_and_shadows_settings_element.md)
- [What's New in Revit 2011 API - MEP Electrical API](./0902-27_mep_electrical_api.md)
- [What's New in Revit 2011 API - Analysis Visualization Framework](./0902-28_analysis_visualization_framework.md)
- [What's New in Revit 2011 API - Family & massing API enhancements](./0902-29_family_massing_api_enhancements.md)
- [What's New in Revit 2011 API - View API changes](./0902-30_view_api_changes.md)
- [What's New in Revit 2011 API - Import, export and print changes](./0902-31_import_export_and_print_changes.md)
- [What's New in Revit 2011 API - Parameter API changes](./0902-32_parameter_api_changes.md)
- [What's New in Revit 2011 API - Alignment and ItemFactoryBase.NewAlignment() method](./0902-33_alignment_and_itemfactorybasenewalignment_method.md)
- [What's New in Revit 2011 API - DimensionType – access to the dimension style](./0902-34_dimensiontype_access_to_the_dimension_style.md)
- [What's New in Revit 2011 API - Create sloped wall on mass face](./0902-35_create_sloped_wall_on_mass_face.md)
- [What's New in Revit 2011 API - Face.GetBoundingBox() method](./0902-36_facegetboundingbox_method.md)
- [What's New in Revit 2011 API - NewFootPrintRoof() input](./0902-37_newfootprintroof_input.md)
- [What's New in Revit 2011 API - Floor slope and elevation](./0902-38_floor_slope_and_elevation.md)
- [What's New in Revit 2011 API - Converting between Model Curves and Detail/Symbolic Curves](./0902-39_converting_between_model_curves_and_detailsymbolic.md)
- [What's New in Revit 2011 API - CurveElement type](./0902-40_curveelement_type.md)
- [What's New in Revit 2011 API - Shared base classes for Room, Space, Area and their tags](./0902-41_shared_base_classes_for_room_space_area_and_their_.md)
- [What's New in Revit 2011 API - View-specific elements](./0902-42_view_specific_elements.md)
- [What's New in Revit 2011 API - Monitoring elements](./0902-43_monitoring_elements.md)
- [What's New in Revit 2011 API - Design Options](./0902-44_design_options.md)
- [What's New in Revit 2011 API - Family template path](./0902-45_family_template_path.md)
- [What's New in Revit 2011 API - Write comments to the Revit Journal File](./0902-46_write_comments_to_the_revit_journal_file.md)
- [What's New in Revit 2011 API - Window extents](./0902-47_window_extents.md)
- [What's New in Revit 2011 API - City.WeatherStation](./0902-48_cityweatherstation.md)
- [What's New in Revit 2011 API - Labels matching commonly used enumerated type values](./0902-49_labels_matching_commonly_used_enumerated_type_valu.md)
- [What's New in Revit 2011 API - Keyboard shortcut support for API buttons](./0902-50_keyboard_shortcut_support_for_api_buttons.md)
- [What's New in Revit 2011 API - MEP API](./0902-51_mep_api.md)
- [What's New in Revit 2011 API - Structural API](./0902-52_structural_api.md)
