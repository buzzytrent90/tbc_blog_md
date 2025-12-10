---
post_number: "0680"
title: "Intermediate API Update Releases"
slug: "update_release_issues"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'revit-api', 'rooms']
source_file: "0680_update_release_issues.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0680_update_release_issues.html"
---

### Intermediate API Update Releases

I recently mentioned the availability of the
[Revit 2012 Update Release 2](http://thebuildingcoder.typepad.com/blog/2011/10/product-and-add-in-wizard-updates.html#2) and
my surprise that some people prefer not to update, which gave rise to a couple of vehement
[comments](http://thebuildingcoder.typepad.com/blog/2011/10/product-and-add-in-wizard-updates.html#comments).

I myself seldom ran into issues with incompatibilities between intermediate releases, but now one affecting the API cropped up:

**Question:** Using the IUpdater interface, I can do the following with the Revit 2012 API:
```csharp
  ElementId paramId = new ElementId(
    BuiltInParameter.ROOM\_NUMBER );

  UpdaterRegistry.AddTrigger(
    GetUpdaterId(), elementFilter,
    Element.GetChangeTypeParameter( paramId ) );
```

The above works for Revit 2012.

In the Revit 2011 API, however, the Element.GetChangeTypeParameter is not overloaded to take the above paramId.
Is there an equivalent way of achieving the same thing with the Revit 2011 API?
I am unable to find a single example for this in the Revit Help, online, or anywhere.

**Answer:** Although this method overload was not provided in the initial release of Revit 2011, it was added by the
[web update 2](http://thebuildingcoder.typepad.com/blog/2010/09/revit-2011-web-update-2.html).

I am not aware of any reason not to update, although some users never do, possibly due to concerns that regressions will halt their current workflow, or just the amount of time it takes to deploy the new version.

As a developer selling your add-in, you need to be conscious of this.
If you do require functionality from a later update release in your application, you should make
[appropriate version checks](http://wikihelp.autodesk.com/Revit/enu/2012/Help/API_Dev_Guide/0001-Introduc1/0028-Applicat28/0029-Applicat29/How_to_use_Application_properties_to_enforce_a_correct_version_for_your_add-in).

This is also one reason why we avoid significant API additions in intermediate update releases, as it becomes more difficult for developers and users of these applications to manage.