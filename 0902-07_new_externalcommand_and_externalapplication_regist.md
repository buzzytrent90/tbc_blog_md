---
author: Jeremy Tammik
optimization_date: '2025-12-11T11:24:06.640574'
original_url: https://thebuildingcoder.typepad.com/blog/0902_whats_new_2011.html
parent_post: 0902_whats_new_2011.md
part_number: '07'
part_total: '52'
post_number: 0902
series: whats_new_in_revit_api
slug: whats_new_2011_new_externalcommand_and_externalapplication_registration_mechanism
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
title: What's New in Revit 2011 API - New ExternalCommand and ExternalApplication
  registration mechanism
word_count: 1300
---

### New ExternalCommand and ExternalApplication registration mechanism

The Revit API now offers the ability to register API applications via a .addin manifest file.

Manifest files will be read automatically by Revit when they are places in one of two locations on a user's system:

- In a non-user specific location in "application data"
  - For Windows XP – C:\Documents and Settings\All Users\Application Data\Autodesk\Revit\Addins\2011\- For Vista/Windows 7 – C:\ProgramData\Autodesk\Revit\Addins\2011\- In a user specific location in "application data"
    - For Windows XP – C:\Documents and Settings\[user]\Application Data\Autodesk\Revit\Addins\2011\- For Vista/Windows 7 – C:\Users\[user]\AppData\Roaming\Autodesk\Revit\Addins\2011\

All files named .addin in these locations will be read and processed by Revit during startup.

A basic file adding one ExternalCommand looks like this:
```csharp
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<RevitAddIns>
  <AddIn Type="Command">
    <Assembly>c:\MyProgram\MyProgram.dll</Assembly>
    <AddInId>76eb700a-2c85-4888-a78d-31429ecae9ed</AddInId>
    <FullClassName>Revit.Samples.SampleCommand</FullClassName>
    <Text>Sample command</Text>
    <VisibilityMode>NotVisibleInFamily</VisibilityMode>
    <VisibilityMode>NotVisibleInMEP</VisibilityMode>
    <AvailabilityClassName>Revit.Samples.SampleAccessibilityCheck</AvailabilityClassName>
    <LongDescription>
      <p>This is the long description for my command.</p>
      </p/>
      <p>This is another descriptive paragraph, with notes about how to use the command properly.</p>
    </LongDescription>
    <TooltipImage>c:\MyProgram\Autodesk.jpg</TooltipImage>
    <LargeImage>c:\MyProgram\MyProgramIcon.png</LargeImage>
  </AddIn>
</RevitAddIns>
```

A basic file adding one ExternalApplication looks like this:
```csharp
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<RevitAddIns>
  <AddIn Type="Application">
    <Name>SampleApplication</Name>
    <Assembly>c:\MyProgram\MyProgram.dll</Assembly>
    <AddInId>604B1052-F742-4951-8576-C261D1993107</AddInId>
    <FullClassName>Revit.Samples.SampleApplication</FullClassName>
  </AddIn>
</RevitAddIns>
```

Multiple AddIn elements may be provided in a single manifest file.

The new mechanism currently offers the following XML tags:

- Assembly – The full path to the add-in assembly file. Required for all ExternalCommands and ExternalApplications.- FullClassName – The full name of the class in the assembly file which implements IExternalCommand or IExternalApplication. Required for all ExternalCommands and ExternalApplications.- ClientId – A GUID which represents the id of this particular application. ClientIds must be unique for a given session of Revit. Autodesk recommends you generate a unique GUID for each registered application or command. Required for all ExternalCommands and ExternalApplications. The property UIApplication.ActiveAddInId provides programmatic access to this value, if required.- Name – The name of application. Required; for ExternalApplications only.- Text – The name of the button. Optional; use this tag for ExternalCommands only. The default is "External Tool".- Description – Short description of the command, will be used as the button tooltip. Optional; use this tag for ExternalCommands only. The default is a tooltip with just the command text.- VisibilityMode – Provides the ability to specify if the command is visible in project documents, family documents, or no document at all. Also provides the ability to specify the discipline(s) where the command should be visible. Multiple values may be set for this option. Optional; use this tag for ExternalCommands only. The default is to display the command in all modes and disciplines, including when there is no active document. Previously written external commands which need to run against the active document should either be modified to ensure that the code deals with invocation of the command when there is no active document, or apply the NotVisibleWhenNoActiveDocument mode.- AvailabilityClassName – The full name of the class in the assembly file which implemented IExternalCommandAvailability. This class allows the command button to be selectively grayed out depending on context. Optional; use this tag for ExternalCommands only. The default is a command that is available whenever it is visible.- LargeImage – The path to the icon to use for the button in the External Tools pulldown menu. The icon should be 32 x 32 pixels for best results. Optional; use this tag for ExternalCommands only. The default is to show a button without an icon.- LongDescription – Long description of the command, will be used as part of the button's extended tooltip. This tooltip is shown when the mouse hovers over the command for a long amount of time. You can split the text of this option into multiple paragraphs by placing <p> tags around each paragraph. Optional; use this tag for ExternalCommands only. If neither of this property and TooltipImage are supplied, the button will not have an extended tooltip.- TooltipImage – The path to an image file to show as a part of the button extended tooltip, shown when the mouse hovers over the command for a longer amount of time. Optional; use this tag for ExternalCommands only. If neither of this property and TooltipImage are supplied, the button will not have an extended tooltip.- LanguageType – Localization setting for Text, Description, LargeImage, LongDescription, and TooltipImage of external tools buttons. Revit will load the resource values from the specified language resource dll. The value can be one of the eleven languages supported by Revit. If no LanguageType is specified, the language resource which the current session of Revit is using will be automatically loaded. For more details see the section on Localization.

The Revit.ini registration mechanism remains in place for the 2011 release but will be removed in the future. The Revit.ini mechanism does not offer any of the new capabilities listed above. In addition, because Dynamic Model Update registration requires a valid AddInId, updaters may not be registered from applications declared in Revit.ini.

#### Localization

You can let Revit localize the user-visible resources of an external command button (including Text, large icon image, long and short descriptions and tooltip image). You will need to create a .NET Satellite DLL which contains the strings, images, and icons for the button. Then change the values of the tags in the .addin file to correspond to the names of resources in the Satellite dll, but prepended with the 'at' character '@'. So the tag:
```csharp
<Text>Extension Manager</Text>
```

Becomes
```csharp
<Text>@ExtensionText</Text>
```

where ExtensionText is the name of the resource found in the Satellite DLL.

The Satellite DLLs are expected to be in a directory with the name of the language of the language-culture, such as en or en-US. The directory should be located in the directory that contains the add-in assembly.
Please refer to
[how to create managed Satellite DLLs](http://msdn.microsoft.com/en-us/library/sb6a8618.aspx).

You can force Revit to use a particular language resource DLL, regardless of the language of the Revit session, by specifying the language and culture explicitly with a LanguageType tag. For example:
```csharp
<LanguageType>English\_USA</LanguageType>
```

would force Revit to always load the values from the en-US Satellite DLL and to ignore the current Revit language and culture settings when considering the localizable members of the external command manifest file.

Revit supports the 11 languages defined in the Autodesk.Revit.ApplicationServices.LanguageType enumerated type: English\_USA, German, Spanish, French, Italian, Dutch, Chinese\_Simplified, Chinese\_Traditional, Japanese, Korean, and Russian.

#### External Command Accessibility

The interface

- IExternalCommandAvailability

allows you control over whether or not an external command button may be pressed. The IsCommandAvailable interface method passes the application and a set of categories matching the categories of selected items in Revit to your implementation. The typical use would be to check the selected categories to see if they meet the criteria for your command to be run.

In this example accessibility check allows a button to be clicked when there is no active selection, or when at least one wall is selected:
```python
public class SampleAccessibilityCheck : IExternalCommandAvailability
{
  public bool IsCommandAvailable(
    Autodesk.Revit.ApplicationServices.Application data,
    CategorySet selectedCategories )
  {
    // Allow button click if there is no active selection
    if( selectedCategories.IsEmpty )
      return true;

    // Allow button click if there is at least one wall selected
    foreach( Category c in selectedCategories )
    {
      if( c.Id.IntegerValue == (int)BuiltInCategory.OST\_Walls )
        return true;
    }
    return false;
  }
}
```
---

## Related Sections
- [What's New in Revit 2011 API - Changes to the Revit API namespaces](./0902-02_changes_to_the_revit_api_namespaces.md)
- [What's New in Revit 2011 API - Split of Revit API DLL](./0902-03_split_of_revit_api_dll.md)
- [What's New in Revit 2011 API - New classes for XYZ, UV, and ElementId](./0902-04_new_classes_for_xyz_uv_and_elementid.md)
- [What's New in Revit 2011 API - Replacement for Symbol and properties that access types](./0902-05_replacement_for_symbol_and_properties_that_access_.md)
- [What's New in Revit 2011 API - New transaction interfaces](./0902-06_new_transaction_interfaces.md)
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
