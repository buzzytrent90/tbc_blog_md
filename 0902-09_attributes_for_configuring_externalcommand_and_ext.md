---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:06.643813'
original_url: https://thebuildingcoder.typepad.com/blog/0902_whats_new_2011.html
parent_post: 0902_whats_new_2011.md
part_number: 09
part_total: '52'
post_number: 0902
series: whats_new_in_revit_api
slug: whats_new_2011_attributes_for_configuring_externalcommand_and_externalapplication_behavior
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
title: What's New in Revit 2011 API - Attributes for configuring ExternalCommand and
  ExternalApplication behavior
word_count: 964
---

### Attributes for configuring ExternalCommand and ExternalApplication behavior

#### Transaction mode

The custom attribute Autodesk.Revit.Attributes.TransactionMode should be applied to your implementation class of the IExternalCommand interface to control transaction behavior for external command. There is no default for this option. You must apply it to legacy application classes to allow your application to function in Revit 2011.

This mode controls how the API framework expects transactions to be used when the command is invoked. There are three supported values:

1. TransactionMode.Automatic – The API framework will create a transaction on the active document before the external command is executed and the transaction will be committed or rolled back after the command is completed (based upon the return value of the ExternalCommand callback). This means that command code cannot create and start its own Transactions, but it can create SubTransactions as required during the implementation of the command. The command must report its success or failure status via the Result return value.- TransactionMode.Manual – The API framework will not create a transaction (but it will create an outer group to roll back all changes if the external command returns a failure status). Instead, you may use combinations of Transactions, SubTransactions, and TransactionGroups as you please. You will have to follow all rules regarding use of transactions and related classes. You will have to give your transaction(s) names, which will then appear in the Undo menu. Revit will check that all transactions (also groups and sub-transactions) are properly closed upon return from an external command. If not, it will discard all changes made to the model.- TransactionMode.ReadOnly – No transaction (nor group) will be created, and no transaction may be created for the lifetime of the command. The External command may use methods that only read from the model, but not methods that write anything to it. Exceptions will be thrown if the command either tries to start a transaction (or group) or attempts to write to the model.

In all three modes, the TransactionMode applies only to the active document. You may open other documents during the course of the command, and you may have complete control over the creation and use of Transactions, SubTransactions, and TransactionGroups on those other documents (even in ReadOnly mode).

For example, to set an external command to use automatic transaction mode:
```csharp
  [Regeneration( RegenerationOption.Manual )]
  [Transaction( TransactionMode.Automatic )]
  public class Command : IExternalCommand
  {
    public Autodesk.Revit.IExternalCommand.Result
      Execute(
        Autodesk.Revit.ExternalCommandData commandData,
        ref string message,
        Autodesk.Revit.ElementSet elements )
    {
      // Command implementation, which modifies the
      // active document directly and no need to
      // start/commit any transactions.
    }
  }
```

#### Regeneration option

The custom attribute Autodesk.Revit.Attributes.RegenerationAttribute should be applied to your implementation class of the IExternalCommand interface and IExternalApplication interface to control the regeneration behavior for the external command and external application. There is no default for this option. You must apply it to legacy application classes to allow your application to function in Revit 2011.

This mode controls whether or not the API framework automatically regenerates after every model modification. There are two supported values:

1. RegenerationOption.Automatic – The API framework will regenerate after every model level change (equivalent behavior with Revit 2010 and earlier). Regeneration and update can be suspended using SuspendUpdating for some operations, but in general the performance of multiple modifications within the same file will be slower than RegenerationOption.Manual. This mode is provided for behavioral equivalence with Revit 2010 and earlier; it is obsolete and will be removed in a future release.- RegenerationOption.Manual – The API framework will not regenerate after every model level change. Instead, you may use the regeneration APIs to force update of the document after a group of changes. SuspendUpdating blocks are unnecessary and should not be used. Performance of multiple modifications of the Revit document should be faster than RegenerationOption.Automatic. Because this mode suspends all updates to the document, your application should not read data from the document after it has been modified until the document has been regenerated, or it runs the risk of accessing stale data. This mode will be only option in a future release.

For example, to set an external command to use manual regeneration mode:
```csharp
  [Regeneration( RegenerationOption.Manual )]
  [Transaction( TransactionMode.Automatic )]
  public class Command : IExternalCommand
  {
    public Autodesk.Revit.IExternalCommand.Result
      Execute(
        Autodesk.Revit.ExternalCommandData commandData,
        ref string message,
        Autodesk.Revit.ElementSet elements )
    {
      // Command implementation, which modifies the
      // document and calls regeneration APIs when
      // needed.
    }
  }
```

To set an external application to use automatic mode
```csharp
  [Regeneration( RegenerationOption.Automatic )]
  public class Application : IExternalApplication
  {
    public Autodesk.Revit.UI.Result OnStartup(
      ControlledApplication a )
    {
      // OnStartup implementation
    }
    public Autodesk.Revit.UI.Result OnShutdown(
      ControlledApplication a )
    {
      // OnShutdown implementation
    }
  }
```

The regeneration mode used for code executed during events and updater callbacks will be the same mode applied to the ExternalApplication or ExternalCommand during which the event or updater was registered.

#### Manual regeneration and update

- Document.Regenerate- Document.AutoJoinElements

These new methods have been added to allow an application running in manual regeneration mode to update the elements in the Revit document at any time.

#### Journaling mode

The custom attribute Autodesk.Revit.Attributes.JournalingAttribute can optionally be applied to your implementation class of the IExternalCommand interface to control the journaling behavior during the external command execution session.

- JournalMode.UsingCommandData

Uses the “StringStringMap” supplied in the command data. Hides all Revit journal entries in between the external command invocation and the StringStringMap entry. Commands which invoke the Revit UI for selection or for responses to task dialogs may not replay correctly.

- JournalMode.NoCommandData

Does not write contents of the ExternalCommandData.Data map to the Revit journal. But does allow Revit API calls to write to the journal as needed. This option should allow commands which invoke the Revit UI for selection or for responses to task dialogs to replay correctly.
---

## Related Sections
- [What's New in Revit 2011 API - Changes to the Revit API namespaces](./0902-02_changes_to_the_revit_api_namespaces.md)
- [What's New in Revit 2011 API - Split of Revit API DLL](./0902-03_split_of_revit_api_dll.md)
- [What's New in Revit 2011 API - New classes for XYZ, UV, and ElementId](./0902-04_new_classes_for_xyz_uv_and_elementid.md)
- [What's New in Revit 2011 API - Replacement for Symbol and properties that access types](./0902-05_replacement_for_symbol_and_properties_that_access_.md)
- [What's New in Revit 2011 API - New transaction interfaces](./0902-06_new_transaction_interfaces.md)
- [What's New in Revit 2011 API - New ExternalCommand and ExternalApplication registration mechanism](./0902-07_new_externalcommand_and_externalapplication_regist.md)
- [What's New in Revit 2011 API - External Command and External Application registration utility](./0902-08_external_command_and_external_application_registra.md)
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
