---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:11.697701'
original_url: https://thebuildingcoder.typepad.com/blog/1428_whats_new_2017.html
parent_post: 1428_whats_new_2017.md
part_number: '37'
part_total: '43'
post_number: '1428'
series: geometry
slug: whats_new_2017_ui_api_additions
source_file: 1428_whats_new_2017.md
tags:
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
- windows
title: API Changes - UI API additions
word_count: 367
---

### UI API additions
#### ColorSelectionDialog
The new class:
- ColorSelectionDialog
provides the option to launch the Revit Color dialog to prompt the user to select a colour. The original colour can be set as well as the selected colour before the user changes it. The method:
- ColorSelectionDialog.Show()
returns a status indicating if a colour was selected and the dialog confirmed, or if the user canceled the selection.
#### FileOpenDialog and FileSaveDialog
The new classes:
- FileOpenDialog
- FileSaveDialog
allow an add-in to prompt the user with the Revit dialog used to navigate to and select an existing file path. FileOpenDialog is typically used to select a file for opening or importing. FileSaveDialog is typically used to enter a file name for saving or exporting.
The behaviour and appearance of this dialog matches the Revit "Open" dialog. This is a general-purpose dialog for opening any given file type, and options to configure settings like worksharing options will not be included. Use of this dialog does not actually open an existing file, but it will provide the selected file path back to the caller to take any action necessary.
These dialogs inherit from:
- FileDialog
which exposes the shared options and operations needed for prompting with either an open or a save dialog. The method:
- FileDialog.Show()
returns a status indicating if a file was selected and the dialog confirmed, or if the user canceled the selection.
#### TaskDialog API additions
The new members:
- TaskDialog.ExtraCheckBoxText
- TaskDialog.WasExtraCheckBoxChecked()
provide access to an extra checkbox shown the user in the TaskDialog. If it is set, a checkbox with the text will be shown in the task dialog. The caller can get the user setting for the checkbox by checking the return value of the WasExtraCheckBoxChecked() method
#### Support for journal data in overridden commands
The new members:
- BeforeExecutingEventArgs.UsingCommandData
- ExecutedEventArgs.GetJournalData()
- ExecutedEventArgs.SetJournalData()
support the ability for the add-in to store journal data associated to an overridden command, similar to the capability offered for External Commands.
#### Dockable Pane API additions
The new members:
- DockablePaneProviderData.VisibleByDefault
support the ability for the add-in to control the whether or not any Dockable Panes they register should be visible by default or not. Default is to true.
---

## Related Sections
- [API Changes - What's New in the Revit 2017 API](./1428-01_whats_new_in_the_revit_2017_api.md)
- [API Changes - Major Revit API Changes and Renovations](./1428-02_major_revit_api_changes_and_renovations.md)
- [API Changes - API Changes](./1428-03_api_changes.md)
- [API Changes - .NET 4.6](./1428-04_net_46.md)
- [API Changes - Visual C++ Redistributable for Visual Studio 2015](./1428-05_visual_c_redistributable_for_visual_studio_2015.md)
- [API Changes - Automatic transaction mode obsolete](./1428-06_automatic_transaction_mode_obsolete.md)
- [API Changes - Code signing of Revit Addins](./1428-07_code_signing_of_revit_addins.md)
- [API Changes - Background processes can load DB applications](./1428-08_background_processes_can_load_db_applications.md)
- [API Changes - Application API changes](./1428-09_application_api_changes.md)
- [API Changes - Family API changes](./1428-10_family_api_changes.md)
- [API Changes - View API changes](./1428-11_view_api_changes.md)
- [API Changes - Text API changes](./1428-12_text_api_changes.md)
- [API Changes - Event API changes](./1428-13_event_api_changes.md)
- [API Changes - Alignment API changes](./1428-14_alignment_api_changes.md)
- [API Changes - Geometry API changes](./1428-15_geometry_api_changes.md)
- [API Changes - Structure API changes](./1428-16_structure_api_changes.md)
- [API Changes - MEP API changes](./1428-17_mep_api_changes.md)
- [API Changes - EnergyDataSettings API changes](./1428-18_energydatasettings_api_changes.md)
- [API Changes - Rendering API changes](./1428-19_rendering_api_changes.md)
- [API Changes - Plane API changes](./1428-20_plane_api_changes.md)
- [API Changes - DirectShape API changes](./1428-21_directshape_api_changes.md)
- [API Changes - Point Cloud API changes](./1428-22_point_cloud_api_changes.md)
- [API Changes - Schedule API changes](./1428-23_schedule_api_changes.md)
- [API Changes - UI API change](./1428-24_ui_api_change.md)
- [API Changes - Obsolete API removal](./1428-25_obsolete_api_removal.md)
- [API Changes - API Additions](./1428-26_api_additions.md)
- [API Changes - Application API additions](./1428-27_application_api_additions.md)
- [API Changes - Family API additions](./1428-28_family_api_additions.md)
- [API Changes - View API additions](./1428-29_view_api_additions.md)
- [API Changes - Text API additions](./1428-30_text_api_additions.md)
- [API Changes - Geometry API additions](./1428-31_geometry_api_additions.md)
- [API Changes - Parameter API additions](./1428-32_parameter_api_additions.md)
- [API Changes - CurveElement API additions](./1428-33_curveelement_api_additions.md)
- [API Changes - Railing API additions](./1428-34_railing_api_additions.md)
- [API Changes - Schedule API additions](./1428-35_schedule_api_additions.md)
- [API Changes - Tag API additions](./1428-36_tag_api_additions.md)
- [API Changes - Structure API additions](./1428-38_structure_api_additions.md)
- [API Changes - Fabrication API additions](./1428-39_fabrication_api_additions.md)
- [API Changes - Electrical API additions](./1428-40_electrical_api_additions.md)
- [API Changes - Other MEP API additions](./1428-41_other_mep_api_additions.md)
- [API Changes - Revit Link API additions](./1428-42_revit_link_api_additions.md)
- [API Changes - Category API additions](./1428-43_category_api_additions.md)
