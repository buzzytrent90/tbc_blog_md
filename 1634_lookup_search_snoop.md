---
post_number: "1634"
title: "Lookup Search Snoop"
slug: "lookup_search_snoop"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'parameters', 'references', 'revit-api', 'selection', 'sheets', 'transactions', 'views']
source_file: "1634_lookup_search_snoop.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1634_lookup_search_snoop.html"
---

### RevitLookup Search by Element and Unique Id
Александр Пекшев aka Modis [@Pekshev](https://github.com/Pekshev) recently
submitted a pull request
to [snoop the stable representation of references](http://thebuildingcoder.typepad.com/blog/2018/03/export-geometry-and-snoop-stable-representation-of-reference.html#2)
for [RevitLookup](https://github.com/jeremytammik/RevitLookup).
Now he implemented another useful enhancement to search and snoop elements by element id or unique id:
- [Search and snoop by element id or unique id](#2)
- [File changes](#3)
- [The built-in Select by Id command, Zoom To and StringSearch](#4)
- [RevitLookup update](#5)
#### Search and Snoop by Element Id or Unique Id
Alexander's RevitLookup [pull request #42 adds a 'search by and snoop' command](https://github.com/jeremytammik/RevitLookup/pull/42):
Add "Search by and snoop" command that allows you to search and snoop for elements by condition:
![Search and Snoop command](img/revitlookup_search_snoop_cmd.png)
A small addition to the project: search for items by `ElementId` or `UniqueId` and then snoop them.
Search options implemented in the form of a drop-down list provide for the possibility of further future expansion (for example, search by parameters or something similar):
![Search and Snoop form](img/revitlookup_search_snoop_form.png)
This view option can be useful for developers who receive an `ElementId` while debugging an application and need to learn about it later in Revit.
Thank you very much, Alexander, for this great functionality added with minimal implementation effort, as you can see by looking and
the [pull request files changed](https://github.com/jeremytammik/RevitLookup/pull/42/files).
#### File Changes
Besides the module \*RevitLookup\CS\Snoop\Forms\SearchBy.cs\* defining the new form, the only changes are the following to add the new menu entry and implement the new external command:
In App.cs:
```csharp
optionsBtn.AddPushButton(new PushButtonData(
"Search and Snoop...",
"Search and Snoop...",
ExecutingAssemblyPath, "
RevitLookup.CmdSearchBy"));
```
In TestCmds.cs:
```csharp
using RevitLookup.Snoop.Forms;
. . .
///
/// Search by and Snoop command: Browse
/// elements found by the condition
/// summary>
[Transaction( TransactionMode.Manual )]
[Regeneration( RegenerationOption.Manual )]
public class CmdSearchBy : IExternalCommand
{
public Result Execute(
ExternalCommandData cmdData,
ref string msg,
ElementSet elems )
{
Result result;
try
{
Snoop.CollectorExts.CollectorExt.m_app = cmdData.Application;
UIDocument revitDoc = cmdData.Application.ActiveUIDocument;
Document dbdoc = revitDoc.Document;
Snoop.CollectorExts.CollectorExt.m_activeDoc = dbdoc; // TBD: see note in CollectorExt.cs
SearchBy searchByWin = new SearchBy( dbdoc );
ActiveDoc.UIApp = cmdData.Application;
searchByWin.ShowDialog();
result = Result.Succeeded;
}
catch( System.Exception e )
{
msg = e.Message;
result = Result.Failed;
}
return result;
}
}
```
#### The Built-in Select by Id Command, Zoom To and StringSearch
Revit does already provide a built-in command that you should be aware of providing similar functionality, at least for selecting an element by element id: Manage > Inquiry > Select by ID > Show:
![Select by ID command](img/select_by_id_cmd.png)
It prompts you to enter the element id and adds the element to the current selection with the option to zoom to it graphically as well:
![Select by ID form](img/select_by_id_form.png)
Once you have selected an element in this manner, you can use the traditional RevitLookup 'snoop current selection' to explore its data.
The 'zoom to' option is very similar to
the [zoom to selected elements](http://thebuildingcoder.typepad.com/blog/2018/03/switch-view-or-document-by-showing-elements.html#3) functionality
discussed last week, implemented using `ShowElements`.
This could obviously be integrated into RevitLookup as well, if so desired.
Another useful piece of functionality that could be integrated is an adaptation of my
old [StringSearch](http://thebuildingcoder.typepad.com/blog/2016/01/all-model-text-stringsearch-2016-and-new-jobs.html) add-in,
originally [ADN Plugin of the Month](http://thebuildingcoder.typepad.com/blog/2011/10/string-search-adn-plugin-of-the-month.html) back
in 2011.
It implements the possibility to search for, list modelessly, select and zoom to elements based on strings in their properties, parameters and other data, including support for regular expressions.
#### RevitLookup Update
Back to here and now, though.
I integrated Alexander's code and fixed some minor compilation errors due to working with Visual Studio 2015.
He presumably used a newer version supporting more permissive C# syntax to create the pull request.
The new functionality is included
in [RevitLookup](https://github.com/jeremytammik/RevitLookup)
[release 2018.0.0.8](https://github.com/jeremytammik/RevitLookup/releases/tag/2018.0.0.8).