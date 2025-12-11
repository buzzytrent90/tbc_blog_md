---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:11.653737'
original_url: https://thebuildingcoder.typepad.com/blog/1428_whats_new_2017.html
parent_post: 1428_whats_new_2017.md
part_number: '01'
part_total: '43'
post_number: '1428'
series: geometry
slug: whats_new_2017_what's_new_in_the_revit_2017_api
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
title: API Changes - What's New in the Revit 2017 API
word_count: 343
---

### What's New in the Revit 2017 API
Here is the online version of the 'What's New' section of the Revit 2017 API help file `RevitAPI.chm` included in the Revit SDK, which is also provided in the separate document '\*Revit Platform API Changes and Additions.docx\*'.
This is very important information, both for exploring the new features and for later reference.
If you encounter any issues at all migrating your add-in between different versions, this is one of the first ports of call.
I am putting it online to ensure that this valuable information is available through web searches.
To look up the introduction of changes and features in previous versions, here are links to the preceding 'What's New in the API' discussions:
- [What's New in the Revit 2010 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2010-api.html)
- [What's New in the Revit 2011 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2011-api.html)
- [What's New in the Revit 2012 API](http://thebuildingcoder.typepad.com/blog/2013/02/whats-new-in-the-revit-2012-api.html)
- [What's New in the Revit 2013 API](http://thebuildingcoder.typepad.com/blog/2013/03/whats-new-in-the-revit-2013-api.html)
- [What's New in the Revit 2014 API](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html)
- [What's New in the Revit 2015 API](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html)
- [What's New in the Revit 2016 API](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html)
For more detailed information, please refer to the complete API documentation and samples collection in the SDK and the
online [Revit API Developers Guide](http://help.autodesk.com/view/RVT/2017/ENU/?guid=GUID-F0A122E0-E556-4D0D-9D0F-7E72A9315A42) provided
as part of the [Revit 2017 Help](http://help.autodesk.com/view/RVT/2017/ENU).
---

## Related Sections
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
- [API Changes - UI API additions](./1428-37_ui_api_additions.md)
- [API Changes - Structure API additions](./1428-38_structure_api_additions.md)
- [API Changes - Fabrication API additions](./1428-39_fabrication_api_additions.md)
- [API Changes - Electrical API additions](./1428-40_electrical_api_additions.md)
- [API Changes - Other MEP API additions](./1428-41_other_mep_api_additions.md)
- [API Changes - Revit Link API additions](./1428-42_revit_link_api_additions.md)
- [API Changes - Category API additions](./1428-43_category_api_additions.md)
