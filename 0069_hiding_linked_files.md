---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:13.311303'
original_url: https://thebuildingcoder.typepad.com/blog/0069_hiding_linked_files.html
post_number: 0069
reading_time_minutes: 3
series: general
slug: hiding_linked_files
source_file: 0069_hiding_linked_files.htm
tags:
- csharp
- elements
- filtering
- revit-api
- views
title: Hiding Linked Files
word_count: 614
---

### Hiding Linked Files

[Rod Howarth](http://roddotnet.blogspot.com)
raised a question in the
[discussion on linked files](http://thebuildingcoder.typepad.com/blog/2008/12/linked-files.html):

**Question:** I'm trying to make a command to turn off the visibility of linked Revit and DWG models in the current view.

The intent is to turn off the visibility for all imported categories in the drawing, i.e. all imported CAD files. This would be like going to Visibility/Graphics > Imported Categories > Show imported categories in this view.

Is there an easy way to do this, or am I going to have to loop over each element, checking if its symbol's name ends in '.dwg' or '.rvt' or is there a certain thing I can look for, i.e. something that tells me it is an imported category, instead of just going by the name?

For DWG, I simply loop through the document settings categories and make each category whose name ends with ".dwg" invisible in the current view.

However, this does not work for RVT, so I had a look with
[RvtMgdDbg](http://download.autodesk.com/media/adn/RvtMgdDbg2009_0429_2008.zip),
and it says that my linked file on the plan is part of the OST\_RvtLinks built-in category.

So I changed my code to do:

```csharp
Document doc = app.ActiveDocument;
Categories categories = doc.Settings.Categories;

// get category for linked files
Category linkedRevitCat
  = categories.get\_Item(
    BuiltInCategory.OST\_RvtLinks );

// loop through all categories in document
foreach( Category c in categories )
{
  // if they end with dwg or rvt toggle
  // their current visibility in current view
  if( c.Name.ToLower().EndsWith( ".dwg" )
    || c.Name.ToLower().Contains( ".rvt" )
    || c.Name.ToLower().EndsWith( ".dwf" )
    || c.Name.ToLower().EndsWith( ".dxf" )
    || c.Name.ToLower().EndsWith( ".dwfx" )
    || ( linkedRevitCat != null
      && c.Id.Equals( linkedRevitCat.Id ) ) )
  {
    // toggle visibility
    doc.ActiveView.setVisibility( c,
      !c.get\_Visible( doc.ActiveView ) );
  }
}
```

Unfortunately, linkedRevitCat always is null, and when I run the elements filter API sample, OST\_RvtLinks doesn't come up as a category list?

**Answer:**
First of all, thank you Rod for raising this interesting question and for pointing out the positive information on how simple it is to hide the linked DWG files.

Joe Ye discovered the following built-in category behaviour:
There is no easy way to filter out those imported categories quickly.
I found one hint that can make your add-in quicker, though:

- For the built-in categories, the element id is always negative.
- For imported ones, which are created after the DWG is imported, the element id is positive.

Using this rule, it is no longer necessary to extract the substring and perform a string comparison for built-in categories.

Phil Xia responded to the question on the OST\_RvtLinks built-in category:
indeed we cannot retrieve the OST\_RvtLinks built-in category from the document settings categories.
We can get it from the element category property, however:

```csharp
  BuiltInCategory bic = BuiltInCategory.OST\_RvtLinks;
  ElementIterator i = doc.Elements;
  Element e;
  while( i.MoveNext() )
  {
    e = i.Current as Element;
    if( e.Name.Contains( ".rvt" )
      && e.Category != null
      && e.Category.Id.Value.Equals( (int) bic ) )
    {
      break;
    }
  }
```

Unfortunately, since we cannot obtain the document category for the RVT link element, it is not possible to hide this category via the View.SetVisibility method like we do for the DWG link element.
Also, unlike the DWG link elements, all the RVT ones return the same category OST\_RvtLinks, so we cannot control the visibility for them individually using this method, even if we were able to obtain the appropriate document category.

I hope these various hints are of some use and will be interested in hearing whether they help resolve your issue, Rod.