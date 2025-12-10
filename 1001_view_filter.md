---
post_number: "1001"
title: "View Filter API"
slug: "view_filter"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'parameters', 'revit-api', 'schedules', 'views']
source_file: "1001_view_filter.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1001_view_filter.html"
---

### View Filter API

One long-standing developer wish list item that was fulfilled by the Revit 2014 API is the ability to access, add and control the view filters listed in the visibility dialogue.

The Revit API now provides the following methods supporting this:

- View.AddFilter()
- View.SetFilterVisibility()
- View.SetFilterOverrides()

Here are several variations of the request for this functionality that help understand the need and new possibilities by highlighting the numerous ways in which it can be used:

**Question 1:** We want to programmatically achieve what can be done using the Filter tab of Visibility/Graphic Override dialogue.

For example, it enables setting the colour of elements in a view that match a filter criterion, e.g., all elements in a view that have a shared parameter value in a certain range can be set to blue.
Other elements in that view with different values can be set to red, all without applying any permanent changes to the model or element attributes.
All the filter, colour and fill pattern objects for achieving this already exist in the API.

We want to hide certain elements as well.
This is also possible in the Filter tab of the Visibility dialogue.
The API provides the View.ProjColorOverrideByElement and View.ProjLineWeightOverrideByElement properties that we currently use, but we still cannot hide elements.

The ElementFilter ViewFilters SDK sample shows how to achieve all the required steps except the final one of assigning a newly created view filter to a view.
In Revit 2013 and before, that can only be done manually through the Graphics/Visibility dialogue and its Filter tab.

**Question 2:** Benson submitted a
[comment](http://thebuildingcoder.typepad.com/blog/2009/11/visible-elements.html?cid=6a00e553e1689788330133f558bdcc970b#comment-6a00e553e1689788330133f558bdcc970b) on
[visible elements](http://thebuildingcoder.typepad.com/blog/2009/11/visible-elements.html) asking for the same thing:

I can select an element, right click it, select Hide in View > By Filter from the pop-up menu and add a filter to hide it.

The method IsHiddenElementOrCategory will still return true for this element, i.e., the element hiding filter is ignored.

I want to obtain all the elements hidden using a filter like this in addition to the global Elements and Category settings.

The classes ParameterFilterElement and FilterRule initially looked as if they might be useful here.
The method FilterRule.ElementPasses can be used to determine if the element is hidden "By Filter", but the filter can be disabled by unchecking it in the view Visibility/Graphic dialogue.

Is it possible to retrieve the filter visibility in a selected view?

I can iterate all the filters and all the rules, but I still see no way to get the visibility of the filters.

**Question 3:** I would like to export only visible objects.
Using the element IsHidden method reports whether it was hidden using the Hide in View > Elements menu option, but ignores Hide in View filters, and so does the
[IsHiddenElementOrCategory](http://thebuildingcoder.typepad.com/blog/2009/11/visible-elements.html) method.

How can my application determine exactly what the user actually sees on the screen?
Our customers obviously expect this functionality for the export.

**Question 4:** I need to programmatically define colour and line type definitions for localisation purposes. This can be achieved through the user interface using Visibility/Graphic Overrides > Filters.

**Question 5:** I need programmatic access to the Visibility/Graphic Overrides filters specifically for Legend views.

**Answer:** The ParameterFilterElement and related classes were renovated to align with the ElementFilter interfaces for element iteration and DMU.

There was initially no direct interface added to support assignment of a filter to a view.

The SCHEDULE\_FILTER\_PARAM and VIS\_GRAPHICS\_FILTERS built-in parameters are possibly used to store these settings, but they cannot be read or modified through the API since they are used to store multiple values, and there is no API support for that kind of value type.

This gap was addressed in the Revit 2014 API, and the documentation highlights the new functionality like this in the
[What's New](http://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html) section:

#### View Filters

A new set of methods on the View class allow getting, setting, adding, and removing filters.
Filters can be created using the ParameterFilterElement class and its Create method that existed in previous versions of the Revit API.

As said, the concrete methods providing access to this include:

- View.AddFilter()
- View.SetFilterVisibility()
- View.SetFilterOverrides()