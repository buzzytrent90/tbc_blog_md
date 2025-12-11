---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.8
content_type: news
optimization_date: '2025-12-11T11:44:13.788497'
original_url: https://thebuildingcoder.typepad.com/blog/0349_migration_steps.html
post_number: 0349
reading_time_minutes: 3
series: general
slug: migration_steps
source_file: 0349_migration_steps.htm
tags:
- csharp
- elements
- filtering
- levels
- references
- revit-api
- transactions
title: Plug-In Migration Steps
word_count: 644
---

### Plug-In Migration Steps

We discussed the
[migration of The Building Coder samples](http://thebuildingcoder.typepad.com/blog/2010/03/porting-the-building-coder-samples-to-revit-2011.html) from
the Revit 2010 to the Revit 2011 API and noted some of the required steps there, but skipped a lot of the obvious and onerous detailed information.
Luckily, my colleague Joe Ye was more patient and precise when he migrated our Revit Structure labs and analysis link sample, and kept track of the required steps in great detail.
So, without further ado, here are Joe's Revit 2011 plug-in project migration notes:

#### Steps to migrate Revit 2010 code to Revit 2011

Before migrating the code, please read the What's New section in the Revit API help file RevitAPI.chm file to know what happened in the API.
Then follow below steps.

1. Open the Revit code project in Visual Studio 2008.- Remove the reference to old Revit API assembly RevitAPI.dll.- Reference the new Revit API assemblies RevitAPI.dll and RevitAPIUI.dll, which are located in the Revit 2011 installation Program subfolder.- Replace old namespaces with the new ones, for example:
         ```csharp
         //using Autodesk.Revit;
         //using Autodesk.Revit.Elements;
         //using Autodesk.Revit.Structural.Enums;
         using Autodesk.Revit.ApplicationServices;
         using Autodesk.Revit.Attributes;
         using Autodesk.Revit.DB;
         using Autodesk.Revit.UI;
         ```
         All namespace mappings are documented in 'Revit 2011 API Namespace Remapping.xlsx' file in SDK.- Decide what regeneration option and transaction mode are required for the external command or application classes.
           This is a non-trivial decision, cf.
           [manual regeneration option risks](http://thebuildingcoder.typepad.com/blog/2010/04/manual-regeneration-mode-danger.html) and
           [best practices](http://thebuildingcoder.typepad.com/blog/2010/04/regeneration-option-best-practices.html).- Add the regeneration option and transaction mode attributes, e.g.
             ```csharp
             [Transaction( TransactionMode.ReadOnly )]
             [Regeneration( RegenerationOption.Manual )]
             ```- If you have used full class names, change the namespace of the IExternalCommand or IExternalApplication interfaces, ExternalCommandData class, and Result enumeration to Autodesk.Revit.UI.- The way to access the Application and Document instances changed.
                 The command data argument now returns a new UIApplication class instance.
                 Use the following code to access the database level application and active document objects:
                 ```csharp
                 UIApplication uiapp = commandData.Application;
                 UIDocument uidoc = uiapp.ActiveUIDocument;
                 Application app = uiapp.Application;
                 Document doc = uidoc.Document;
                 ```
                 In Revit 2010, there was no separation between the user interface and database level application and document objects.
                 The application was accessed through the ExternalCommandData.Application property, and the active document through ExternalCommandData.Application.Document.- If the project has element iteration processes, you need to migrate these to use the new FilteredElementCollector class.
                   Please refer to RevitAPI.chm and Revit 2011 API developer guide.pdf for detailed information.- Some classes were renamed. Please read the What's New section in RevitAPI.chm for detailed information, and replace the old class names with the ones.

#### Other notes

1. Creating a new ElementId instance requires an argument, unlike Revit 2010.- The Document get\_Element( ElementId ) method now takes an ElementId argument without the ref keyword, because ElementId is a class now.- TypeFilter changed to ElementClassFilter. Creating instance change to use New ElementClassFilter- CategoryFilter becomes ElementCategoryFilter. Other element filter class are also renamed.- Access to the Revit Structure analytical model changed.- The ElementId Value property changed to IntegerValue, so replace with the new property name.- When retrieving document elements, we need to set a filter explicitly. If no filter was set at all, an exception is thrown.- If the command is using the manual regeneration option and the code makes changes to the model, we need to explicitly create a transaction to update the model:
                 ```csharp
                 Transaction t = new Transaction(
                 doc, "Test" );
                 t.Start();
                 . . .
                 t.Commit();
                 ```- Some other detailed items are not listed here.
                   Please refer to the API changes documentation for further information.

Many thanks to Joe for these detailed instructions!