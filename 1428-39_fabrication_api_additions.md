---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:11.700402'
original_url: https://thebuildingcoder.typepad.com/blog/1428_whats_new_2017.html
parent_post: 1428_whats_new_2017.md
part_number: '39'
part_total: '43'
post_number: '1428'
series: geometry
slug: whats_new_2017_fabrication_api_additions
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
title: API Changes - Fabrication API additions
word_count: 529
---

### Fabrication API additions
#### FabricationPart – product list support
To specify a size, some FabricationPart elements, such as purchased duct and pipe fittings, have a Product Entry field in the Properties palette. In the API these FabricationPart elements are identified as having a "product list". The product list entries represent a catalog of available sizes for the selected part.
The following new members are added to support product list FabricationPart elements:
- FabricationPart.ProductListEntry – The product list entry index of the fabrication part. A value of -1 indicates the fabrication part is not a product list.
- FabricationPart.IsProductList()
- FabricationPart.GetProductListEntryName()
- FabricationPart.GetProductListEntryCount()
- FabricationPart.IsProductListEntryCompatibleSize() – Checks to see if this part can be changed to the specified product list entry without altering any connected dimensions.
#### Design to fabrication conversion
The new class:
- DesignToFabricationConverter
supports the conversion of design elements to FabricationPart elements. Use the method:
- DesignToFabricationConverter.Convert()
to carry out the conversion, and use the available accessor methods to get the elements created during the conversion, and the elements which failed to convert for various reasons.
#### FabricationPart – Stretch and fit
The new method:
- FabricationPart.StretchAndFit()
supports the operation to stretch the fabrication part from the specified connector and fit to the target routing end. The routing end is indicated as a FabricationPartRouteEnd object, which can be obtained from:
- FabricationPartRouteEnd.CreateFromConnector()
- FabricationPartRouteEnd.CreateFromCenterline()
#### Support for fabrication configuration profiles
The new method:
- FabricationConfiguration.SetConfiguration(Autodesk.Revit.DB.FabricationConfigurationInfo info, System.String profile)
sets the fabrication configuration with the specified profile.
The new methods:
- FabricationConfiguration.GetProfile()
- FabricationConfiguration.GetProfiles()
return the profile associated with the loaded configuration or all configurations, respectively.
#### Support for accessing fabrication configuration data abbreviations added
The new methods:
- FabricationConfiguration.GetMaterialAbbreviation()
- FabricationConfiguration.GetSpecificationAbbreviation()
- FabricationConfiguration.GetInsulationSpecificationAbbreviation()
#### MEP Fabrication API
The following new members and properties have been added to the MEP fabrication and FabricationPart capabilities:
The new properties:
- FabricationPart.ServiceId
- FabricationPart.ServiceName
- FabricationPart.ServiceAbbreviation
- FabricationPart.TopOfPartElevation
- FabricationPart.BottomOfPartElevation
- FabricationPart.ItemNumber
- FabricationPart.ItemCustomId
- FabricationPart.Size
- FabricationPart.Slope
- FabricationPart.OverallSize
- FabricationPart.Weight
- FabricationPart.HasInsulation
- FabricationPart.InsulationType
- FabricationPart.InsulationThickness
- FabricationPart.InsulationArea
- FabricationPart.HasLining
- FabricationPart.LiningType
- FabricationPart.LiningThickness
- FabricationPart.LiningArea
- FabricationPart.SheetMetalArea
- FabricationPart.MaterialThickness
- FabricationPart.HasDoubleWall
- FabricationPart.DoubleWallMaterial
- FabricationPart.DoubleWallMaterialThickness
- FabricationPart.DoubleWallMaterialArea
- FabricationPart.IsBoughtOut
- FabricationPart.Notes
- FabricationPart.Alias
- FabricationPart.Vendor
- FabricationPart.VendorCode
- FabricationPart.ProductName
- FabricationPart.ProductShortDescription
- FabricationPart.ProductLongDescription
- FabricationPart.ProductFinishDescription
- FabricationPart.ProductSpecificationDescription
- FabricationPart.ProductMaterialDescription
- FabricationPart.ProductSizeDescription
- FabricationPart.ProductDataRange
- FabricationPart.ProductOriginalEquipmentManufacture
- FabricationPart.ProductInstallType
The new methods:
- FabricationPart.RotateConnecteFabricationPart.ServiceIddPartByConnector()
- FabricationPart.RotateConnectedTap()
- FabricationPart.StretchAndFit()
- FabricationServiceButton.ContainsFabricationPartType()
- FabricationServiceButton.GetConditionDescription()
- FabricationPart.CreateHanger() – Creates a free placed hanger.
- FabricationPart.Reposition() – Repositions the fabrication straight part to another end of the run.
- FabricationPart.PlaceFittingAsCutIn() – Places the fitting on the straight part by cut in, use the fitting's focal point as the insertion position.
- FabricationPart.CanAdjustEndLength() – Checks if the end of fabrication part can be adjusted.
- FabricationPart.AdjustEndLength() – Adjusts the length for the specified connector of the fabrication part.
- FabricationHostedInfo.GetBearerCenterline() – Gets the centerline of the bearer. This method is applicable only for bearer hangers.
- FabricationRodInfo.SetRodEndPosition() – Sets the position of the rod end.
- FabricationRodInfo.AttachToHanger() – Attaches the hanger rod to another bearer hanger.
- FabricationRodInfo.IsRodLockedWithHost() – Checks if the rod is locked with the host.
- FabricationRodInfo.SetRodLockedWithHost() – Locks the rod with the host.
- FabricationRodInfo.GetBearerExtension() – Gets the bearer extension.
- FabricationRodInfo.SetBearerExtension() – Sets the bearer extension.
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
- [API Changes - UI API additions](./1428-37_ui_api_additions.md)
- [API Changes - Structure API additions](./1428-38_structure_api_additions.md)
- [API Changes - Electrical API additions](./1428-40_electrical_api_additions.md)
- [API Changes - Other MEP API additions](./1428-41_other_mep_api_additions.md)
- [API Changes - Revit Link API additions](./1428-42_revit_link_api_additions.md)
- [API Changes - Category API additions](./1428-43_category_api_additions.md)
