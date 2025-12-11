---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.5
content_type: code_example
optimization_date: '2025-12-11T11:44:13.767785'
original_url: https://thebuildingcoder.typepad.com/blog/0336_addinutility.html
post_number: '0336'
reading_time_minutes: 6
series: general
slug: addinutility
source_file: 0336_addinutility.htm
tags:
- family
- python
- revit-api
- selection
- views
- walls
title: RevitAddInUtility
word_count: 1206
---

### RevitAddInUtility

In a
[comment](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-product-guids.html?cid=6a00e553e1689788330133ec852f18970b#comment-6a00e553e1689788330133ec852f18970b) on
the recent discussion of the
[Revit 2011 product GUIDs](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-product-guids.html),
David Bartliff raised an interesting question on the new Revit add-in utility DLL which triggered some internal discussion as well.

Before addressing David's question, however, let's look at some background info on what this utility is about at all.

#### Revit Add-In Utility DLL

The Revit add-in utility DLL is a stand-alone .NET assembly which offers a dedicated API capable of reading, writing and modifying add-in manifest files.
It is intended for use from product installers and scripts.

The developer guide discusses it briefly in section 3.4.2 and provides two sample code snippets demonstrating 'Creating and editing a manifest file' and 'Reading an existing manifest file'.

The classes defined by the add-in utility DLL with all their member methods and properties are documented in the help file RevitAddInUtility.chm in the SDK installation folder.
The SDK also provides two sample applications which demonstrate its use.

Before we can discuss them fully, there is yet another piece of new valuable Revit 2011 add-in management functionality which we need to mention:

#### The External Command Availability Interface IExternalCommandAvailability

This interface allows you control over whether or not an external command button may be pressed in a given Revit project or family context.
The IsCommandAvailable interface method passes the application and a set of categories matching the currently selected items in Revit to the implementation class.
The typical use would be to check the selected categories to see if they meet the criteria for your command to be run.
Other criteria could obviously also be added, so you can easily define an application that can only be used between ten thirty and eleven o'clock in the morning, or when the moon is full.
Here is an accessibility checking example from the developer guide section 3.2.2.5 which allows a command to be launched when there is no active selection, or when at least one wall is selected:
```python
public class SampleAccessibilityCheck
  : IExternalCommandAvailability
{
  public bool IsCommandAvailable(
    UIApplication applicationData,
    CategorySet selectedCategories )
  {
    // Allow button click if there is
    // no active selection

    if( selectedCategories.IsEmpty )
      return true;

    // Allow button click if there is
    // at least one wall selected

    foreach( Category c in selectedCategories )
    {
      if( c.Id.IntegerValue
        == ( int ) BuiltInCategory.OST\_Walls )
        return true;
    }
    return false;
  }
}
```

I just noticed that the same sample code is also included in the help file, by the way.

#### ExternalCommand2011 SDK Samples

As mentioned in the
[product GUID post](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-product-guids.html),
the Revit 2011 SDK includes a new subfolder named ExternalCommand2011 containing the two applications ExternalComandRegistration and RevitAddInUtilitySample.
They demonstrate how to make efficient use of some of the new Revit add-in utility and external command registration functionality:

- RevitAddInUtility: use the RevitAddInUtility DLL to create and edit an add-in manifest file, read existing data from it, and retrieve the installed Revit product information.- ExternalComandRegistration: new external command registration features:
    1. Visibility Mode: control the visibility of each external command based on the different product and document type.- IAvailabilityClass: enable and disable each external command based on user selection or application information.- Icon and tooltip: define icon and tooltip.- Localisation: localise strings in the add-in manifest file.

#### RevitAddInUtility

Demonstrates how to write a new add-in manifest file, retrieve information from a given manifest file or an entire folder containing manifest files, and how to retrieve information on installed Revit products on the local system.
It displays the following dialogue with three buttons and a tree view to demonstrate various RevitAddInUtility functions:

![RevitAddInUtility](img/RevitAddInUtility.png)

It provides the following functionality:

1. Create a new add-in manifest file with an external application and an external command in it.- Retrieve external command and application information from the newly created add-in manifest and display them in the tree view.- Retrieve information about installed Revit products and corresponding add-in manifest folders and display it in the tree view.

I assume that this sample is the application David is alluding to in his question below.

#### ExternalComandRegistration

Demonstrate user how to register external commands making use of the new features provided in Revit 2011, including:

1. Provides a console application in which we create an add-in manifest which contains two external commands by RevitAddInUtility.- Set 'VisibilityMode' node of these two external commands with different values to demonstrate how to visualize external command base on the document type or project type change. Set visibility mode of first command with 'NotVisibleInStructure | NotVisibleWhenNoActiveDocument', Set visibility mode of second command with 'NotVisibleInMEP | NotVisibleInFamily'.- Provides user an assembly with two classes which inherited from IAvailabilityClass interface:- First class: return false from IsCommandAvailabilable(UIApplication^ app, CategorySet^ categories) when user selected a wall. Use CategorySet property to judge users selection.- Second class: return false from IsCommandAvailabilable(UIApplication^ app, CategorySet^ categories) when current document is in 3D view.- Set 'AvailabilityClass' node of these two external commands with class name provided in step 3.- Provides resource files for both English and Chinese language. Including both picture and text string to demonstrate the localization, icon and tooltip.- Set icon and tooltip image of these two external commands with images above, and set  node as 'English\_USA'. And tell user how to change language to 'Chinese\_Simplified' to display external command by Chinese.- Provides an easy add-on application to bind with each external command.

#### Locating the RevitAddInUtility Assembly

In order to make use of the functionality provided by the RevitAddInUtility assembly, for instance in an installer, the application needs to first locate and load it.
Here is David's
[question](http://thebuildingcoder.typepad.com/blog/2010/03/revit-2011-product-guids.html?cid=6a00e553e1689788330133ec852f18970b#comment-6a00e553e1689788330133ec852f18970b) on this:

**Question:** Will the actual release version of RevitAddInUtility.dll be added to the GAC – it is not for Beta 2.
We built the sample exe and put it on the desktop of a user's PC and it failed to work properly because it could not find the utility DLL.
So we either have to locate the Revit program folder ourselves or use a local copy of the utility DLL – which seems to defeat the purpose somewhat.

**Answer:**

RevitAddinUtility.dll is a stand-alone tool and does not depend on any other Revit modules.
Revit application developers can freely include it in their installers.

Currently, this DLL is located in the product install folder.
The long-term plan is for it to be moved into the SDK.
The intention is to allow redistribution of the DLL with program installers, although there is no legal language about that in the Revit install or SDK at this point.

For ADN members, this information is also provided in the technical solution
[TS87598](http://adn.autodesk.com/adn/servlet/devnote?siteID=4814862&id=9647178&linkID=4901650).

Maybe putting it in the GAC is a good alternate solution which might be feasible if it is indeed entirely managed and has no dependencies whatsoever on Revit DLLs.
Thank you for that suggestion!