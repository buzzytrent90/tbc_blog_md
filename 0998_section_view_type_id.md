---
post_number: "0998"
title: "Language Independent Section View Type Id Retrieval"
slug: "section_view_type_id"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'revit-api', 'schedules', 'views']
source_file: "0998_section_view_type_id.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0998_section_view_type_id.html"
---

### Language Independent Section View Type Id Retrieval

I already discussed
[changing the viewport type](http://thebuildingcoder.typepad.com/blog/2013/01/changing-viewport-type.html) and
mentioned Steve Mycynek's all-encompassing AU class
[CP3133 Using the Revit Schedule and View APIs](http://thebuildingcoder.typepad.com/blog/2012/11/au-classes-on-the-view-mep-and-link-apis.html#2) explaining
everything else there is to know about the topic of generating new views.

Here is an important language independence enhancement for this provided by Håkan Wikemar of
[AEC AB](http://www.aec.se), who asked:

**Question:** I ran into a little problem trying to create some section views, in which a view type element id has to be specified:

```csharp
  NyViewSectionB = ViewSection.CreateSection(
    doc.Document, SectionId, bbNewSectionF );
```

When I try to find a section view family type element id, the section I was looking for was not always available in all projects, so the search failed.

I was making use of the GetElementByName method provided by Steven in his class.

When the section view type is present, it is sometimes not named 'Building Section' as in the example we looked at in the View Class at AU. It could be named 'Section 1' if there is no inserted section and whatever name I specified myself if I already created a section in the project.

Is it at all possible to retrieve the section view type id consistently language independently instead of searching by name?

How should one handle this in a file without any sections available?

I would have preferred a Method just taking the an argument such as: 'ViewType.Section'.

Here are some sample view types from an existing project displayed by RevitLookup snoop:

![Old view types](img/view_types_snoop_old.png)

This is the corresponding list in a new project:

![New view types](img/view_types_snoop_new.png)

**Answer:** How about simply iterating over all view types and picking the first one of ViewType.Section?

**Response:** Yes, absolutely.
The view name can vary depending on project.
I needed a more consistent way of getting a section definition element id.
I implemented the following and it seems to do the trick:

```csharp
  ElementId SectionId = GetViewTypeIdByViewType(
    ViewFamily.Section );

  ViewSection NyViewSectionR
    = ViewSection.CreateSection(
      doc.Document, SectionId, bbNewSectionR );

  ElementId GetViewTypeIdByViewType(
    ViewFamily viewFamily )
  {
    FilteredElementCollector fec
      = new FilteredElementCollector(
        m\_app.ActiveUIDocument.Document );

    fec.OfClass( typeof( ViewFamilyType ) );

    foreach( ViewFamilyType e in fec )
    {
      System.Diagnostics.Debug.WriteLine( e.Name );

      if( e.ViewFamily == viewFamily )
      {
        return e.Id;
      }
    }
    return null;
  }
```

If needed, I could continue the iteration when several suitable types are found and let the user choose, but that has not yet proved necessary.

Many thanks to Håkan for sharing this!