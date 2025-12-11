---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.8
content_type: news
optimization_date: '2025-12-11T11:44:14.360960'
original_url: https://thebuildingcoder.typepad.com/blog/0667_string_search_potm.html
post_number: '0667'
reading_time_minutes: 5
series: filtering
slug: string_search_potm
source_file: 0667_string_search_potm.htm
tags:
- csharp
- elements
- family
- parameters
- revit-api
- selection
- sheets
- views
- walls
- windows
- filtering
title: String Search ADN Plugin of the Month
word_count: 954
---

### String Search ADN Plugin of the Month

Best regards from Andalusia!
![Leaving the beach in Nerja](img/jeremy_in_nerja.jpg)

Here is just a short note to point out that my first ADN Plugin of the Month has been published on Autodesk Labs: the November ADN Plugin of the Month is the
[Revit String Search](http://labs.blogs.com/its_alive_in_the_lab/2011/10/november-adn-plugin-of-the-month-revit-search-string-now-available.html).

Look at the [developer notes](#notes) at the end for reasons why you should take a look at this :)
![String Search](img/string_search_panel.png)

#### Description

Search Revit project elements and their parameter values for a given string value.

Standard string search options can be specified to define how the string is matched, which Revit elements and which of their parameters are searched.

Results are presented in a modeless navigator form listing the search hits. Selecting a search hit zooms to and highlights the Revit element containing it.

#### System Requirements

This plugin has been tested with Revit Architecture 2011 and 2012, and requires the .NET Framework 3.5.

A pre-built version of the plug-in has been provided which works on both Revit 2011 and 2012, and on both the 32- and 64-bit Windows systems.

The source code has been provided as a Visual Studio 2008 C# project. It is not required to run the plugin.

#### Usage

In Revit, click "Add-Ins" > "String Search" panel > "String Search" to launch the command.

In the dialogue that appears, you can type in the search string in the 'Find what' text box.

Click 'OK'. The command will search for the given string on all standard and user parameters of all elements in the current view.

All occurrences of the search string are listed in a data grid view in a modeless navigator dialogue. Double click on a row in the navigator to zoom to the element.

A log file of the search operation and its results is created in your temporary directory. To view the log file, right click on the search form and select 'Display Log File'.

The string matching options and the elements and parameters to search can be modified as follows.
![String Search form](img/string_search_form.png)

#### Options

Category: Limit the search to elements belonging to a specific category. All other categories will be skipped. Specify an asterisk '\*' to search all categories. This is the default setting.

Parameter name: You can specify the name of a specific parameter to search in. All other parameters will be skipped. Specify an asterisk '\*' to search all parameters. This is the default setting.

Find options: Match case and match whole parameter limit the string pattern matching to more specific cases, i.e. the upper and lower case of each character must match, and the specified search string must match the entire parameter value.

Element selection: You can search the currently selected element set, all elements in the current view, or all elements in the entire project. The latter can be slow.

Instances versus Types and Symbols: You can specify whether to search in the BIM elements themselves, such as walls and family instances, or in the element types, such as wall types and family symbols. When searching element types, there is no way to limit the search to selected elements (because they cannot be selected) or to a view (because they are not displayed graphically). By default, all elements in the current view are searched. Searching types is not additional to instances, it is complementary. You can search either element types (derived from ElementType in the API) or instances, but not both at once.

Parameter selection: You can select whether only standard built-in Revit parameters are searched, or user-defined family and shared parameters, or both.

Advanced: You have the option of using a regular expression to specify the search string. This option is intended for users with advanced programming knowledge and is not suited for non-programmers. The .NET Regex library is used for this. For more detail about regular expression, please refer to the one of the available ['cheat sheets'](http://regexlib.com/cheatsheet.aspx).

You can also search in BuiltInParameters, which are only visible through the Revit API. Again, this option is intended for users with advanced programming knowledge and is not suited for others. With this option, the parameter selection option is ignored and the 'Parameter name' setting switches to a list of all built-in parameters. Again, you can search in all of them using the asterisk '\*', or pick a specific one. Searching all built-in parameters across the entire project can be extremely slow.

#### Examples

To search for a string 'abc' in all parameter values of all elements in the current view, simply type in 'abc' as the search string and hit OK. If any occurrences are found, they are listed in the modeless navigator window. You can continue working in Revit as normal. If you double click an entry in the navigator, the Revit screen will zoom in to the selected element and highlight it.

To search for all structurally non-bearing walls, you might make use of the built-in parameter option as follows:

- Enter the search string 'Non-bearing'- Under Category, select Walls- Under Advanced, select 'Search BuiltInParameters'- Under Built-in parameter, select WALL\_STRUCTURAL\_USAGE\_TEXT\_PARAM

Click OK and examine the results in the navigator.

#### Notes for Developers

- Hopefully optimal interaction with modeless search result navigator, as described for the
  [modeless loose connector navigator](http://thebuildingcoder.typepad.com/blog/2011/09/yet-another-modeless-update.html).- Neat external application implementation using built-in image resources.- Handling of duplicate built-in parameter enumeration values.- Identical source code for both Revit 2011 and 2012.