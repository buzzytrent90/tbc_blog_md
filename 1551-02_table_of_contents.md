---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:10.418568'
original_url: https://thebuildingcoder.typepad.com/blog/1551_whats_new_2018.html
parent_post: 1551_whats_new_2018.md
part_number: '02'
part_total: '23'
post_number: '1551'
series: geometry
slug: whats_new_2018_table_of_contents
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
title: API Changes - Table of Contents
word_count: 540
---

### Table of Contents
I implemented
the [Python script `number_html_headings.py` described below](#4) to
generate the HTML heading numbering for this table of contents:
- [API changes](#2)
- [API enabled in perspective views](#2.1)
- [Element modifications and contextual commands](#2.1.1)
- [External commands and applications](#2.1.2)
- [Subelements](#2.2)
- [References and selection of subelements](#2.2.3)
- [Changes to APIs for accessing version](#2.3)
- [Asset API Changes](#2.4)
- [Dynamic Updaters on Reload Latest](#2.5)
- [Export to DWG/DXF API change](#2.6)
- [UIDocument.PromptForFamilyInstancePlacement() behavioral change](#2.7)
- [IndependentTag API changes](#2.8)
- [Shared Coordinates API changes](#2.9)
- [New DirectShape behaviors](#2.10)
- [Rebar API Changes](#2.11)
- [FabricSheet API Changes](#2.12)
- [Structural Section API Changes](#2.13)
- [ElectricalSystem API changes](#2.14)
- [Pipe Pressure Loss Calculation change](#2.15)
- [Corrected names of AutoRouteFailures values](#2.16)
- [Obsolete API removal](#2.17)
- [Classes](#2.17.4)
- [Methods](#2.17.5)
- [Properties](#2.17.6)
- [Enumerated types](#2.17.7)
- [API additions](#3)
- [API to get list of reviewable warnings from a Document](#3.18)
- [API access to FamilyInstance references](#3.19)
- [Multistory Stairs API](#3.20)
- [Railings API additions related to MultistoryStairs](#3.20.8)
- [Dimension API additions](#3.21)
- [DimensionEqualityLabelFormating API](#3.21.9)
- [UnitsFormatOptions in DimensionType](#3.21.10)
- [OrdinateDimensionSetting](#3.21.11)
- [SpatialElementTag API additions](#3.22)
- [Geometry API additions](#3.23)
- [Surface and Face API](#3.23.12)
- [RevolvedSurface API](#3.23.13)
- [RuledSurface API](#3.23.14)
- [Level API addition](#3.24)
- [Dockable Frame API Additions](#3.25)
- [DirectContext3D for display of externally managed 3D graphics in Revit](#3.26)
- [View update for DirectContext3D](#3.26.15)
- [Coordination Model elements](#3.27)
- [Shared Coordinates API additions](#3.28)
- [Acquire and Publish coordinates API additions](#3.28.16)
- [SiteLocation API additions](#3.28.17)
- [ProjectLocation API additions](#3.28.18)
- [Revit Link API additions](#3.28.19)
- [Link API additions](#3.29)
- [External Resource framework additions](#3.29.20)
- [CADLinkType additions](#3.29.21)
- [ImportInstance additions](#3.29.22)
- [IFC Link API additions](#3.29.23)
- [Workshared operation progress changed events](#3.30)
- [WorksharedOperationProgressChangedEventArgs](#3.30.24)
- [DocumentSaveToLocalProgressChangedEventArgs](#3.30.25)
- [DataTransferProgressChangedEventArgs](#3.30.26)
- [DocumentReloadLatestProgressChangedEventArgs](#3.30.27)
- [DocumentSaveToCentralProgressChangedEventArgs](#3.30.28)
- [CreateRelatedFileProgressChangedEventArgs](#3.30.29)
- [Events related to linked resources](#3.31)
- [LinkedResourceOpeningEventArgs](#3.31.30)
- [LinkedResourceOpenedEventArgs](#3.31.31)
- [Events related to parallel View Export](#3.32)
- [DWG export API additions](#3.33)
- [ExportDWGSettings API additions](#3.33.32)
- [Part API additions](#3.34)
- [Freeform Rebar API additions](#3.35)
- [Custom Service for Freeform Rebar Definition](#3.36)
- [HVACSpaceType and Space air ventilation API](#3.37)
- [Energy Analysis API additions](#3.38)
- [Electrical API additions](#3.39)
- [Electrical Circuit Path API](#3.39.33)
- [PanelScheduleView API](#3.39.34)
- [MEPAnalyticalConnection API additions](#3.40)
- [Fabrication API additions](#3.41)
- [Hanger Rod additions](#3.41.35)
- [Placing and splitting fabrication parts](#3.41.36)
- [Fabrication Part export additions](#3.41.37)
- [Fabrication Part status additions](#3.41.38)
- [Fabrication part comparison](#3.41.39)
- [Fabrication Part ancillary usage additions](#3.41.40)
- [Fabrication Part custom data additions](#3.41.41)
- [FabricationConfiguration additions](#3.41.42)
- [Routing exclusions additions](#3.41.43)
- [FabricationServiceButton addition](#3.41.44)
---

## Related Sections
- [API Changes - What's New in the Revit 2018 API](./1551-01_whats_new_in_the_revit_2018_api.md)
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
- [API Changes - View update for DirectContext3D](./1551-19_view_update_for_directcontext3d.md)
- [API Changes - Acquire and Publish coordinates API additions](./1551-20_acquire_and_publish_coordinates_api_additions.md)
- [API Changes - SiteLocation API additions](./1551-21_sitelocation_api_additions.md)
- [API Changes - ProjectLocation API additions](./1551-22_projectlocation_api_additions.md)
- [API Changes - Revit Link API additions](./1551-23_revit_link_api_additions.md)
