---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.2
content_type: tutorial
optimization_date: '2025-12-11T11:44:13.454814'
original_url: https://thebuildingcoder.typepad.com/blog/0155_category_comparison_2.html
post_number: '0155'
reading_time_minutes: 4
series: elements
slug: category_comparison_2
source_file: 0155_category_comparison_2.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- revit-api
- selection
- views
title: Category Comparison and Model Elements Revisited
word_count: 896
---

### Category Comparison and Model Elements Revisited

This is a follow-up to two previous posts, on
[category comparison](http://thebuildingcoder.typepad.com/blog/2009/01/category-comparison.html)
and the
[selection of model elements](http://thebuildingcoder.typepad.com/blog/2009/05/selecting-model-elements.html).

Regarding the
[category comparison](http://thebuildingcoder.typepad.com/blog/2009/01/category-comparison.html),
we discussed the fact that categories cannot be compared directly, and one should use their element id instead.
We also underlined that every category comparison should be made language independent by using the built-in category enumeration and not comparing the localised category names.
The analysis of the
[model element selection](http://thebuildingcoder.typepad.com/blog/2009/05/selecting-model-elements.html)
examined how to eliminate the preview legend components from the selected elements, since they suddenly acquired geometry in Revit 2010 and also fit the other criteria applied in the selection process.

As Miroslav Schonauer pointed out, the model element selection that I suggested in that post will not work, because various model elements exist that do not have a valid level assigned to them.
He suggests that the previous selection algorithm with an additional check added for the preview legend components probably remains the simplest and most reliable method.

One deplorable aspect of the code presented there with the preview legend component check added is its language dependence.
It compares the element category name with the preview legend component one, which is "Legend Components".
I left that in there because I thought I would use the alternative approach anyway, checking for a valid level.

So now the issue is how to compare a given element's category against a built-in one without using the category name.
In previous similar situations, I achieved this by asking the document for the built-in category object and using its element id.
The Revit developers guide includes some sample code which demonstrates an extremely effective alternative method.
It avoids the string comparison and achieves language independence, and it also does not require the effort of querying the document settings for the element id of a built-it category.

For built-in categories, the value of the element id is exactly the same as the integer resulting from casting the built-in category enumeration value.
The section 5.2.1 Category discusses details of the element category classification and explains:

An element's category is determined by the Category ID.

- Category IDs are represented by the ElementId class.- Imported Category IDs correspond to elements in the document.- Most categories are built-in and their IDs are constants stored in ElementIds.- Each built-in category ID has a corresponding value in the BuiltInCategory Enumeration.
        They can be converted to corresponding BuiltInCategory enumerated types.- If the category is not built-in, the ID is converted to a null value.

The Code Region 14-1: 'Distinguishing permanent dimensions from constraints' in section 14.1 'Dimensions and Constraints' shows that we can actually use the built-in enum value to compare with an element category id directly, as in the following excerpted lines:

```csharp
  if ( (int) BuiltInCategory.OST\_Dimensions
    == dimension.Category.Id.Value )
  {
    message += "\nDimension is a permanent dimension.";
  }
```

Making use of this for the model element selection, we can implement the following filter which is both language independent and efficient:

```csharp
BuiltInCategory \_bicPreviewLegendComponent
  = BuiltInCategory.OST\_PreviewLegendComponents;

public CmdResult Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  ElementIterator it = doc.Elements;
  string s = string.Empty;
  int count = 0;

  Geo.Options opt = app.Create.NewGeometryOptions();

  int iBic = (int) \_bicPreviewLegendComponent;

  while( it.MoveNext() )
  {
    Element e = it.Current as Element;

    if ( !(e is Symbol)
      && !(e is FamilyBase)
      && (null != e.Category)
      && ( iBic != e.Category.Id.Value )
      && (null != e.get\_Geometry( opt )) )
    {
      ++count;
      s += string.Format(
        "\r\n  Category={0}; Name={1}; Id={2}",
        e.Category.Name, e.Name,
        e.Id.Value.ToString() );
    }
  }

  s = "There are " + count.ToString()
    + " model elements:" + s;

  LabUtils.InfoMsg( s );

  return CmdResult.Failed;
}
```

Here is a snapshot of an updated version of the
[Revit API introduction labs](zip/rac_labs_2009-06-19.zip),
including the updated version of Lab 2-2 to select all model elements using this criterion and other on-going improvements we have made.
In particular, it includes two completely new modules, Labs6 dealing with external application and event issues, and Labs7 demonstrating the form creation API.
We are still continuing the process of updating and enhancing them for the Revit 2010 API and its new features.
All of the labs are provided in two versions, one each for C# and VB.
This version also includes a set of HTML instructions for self-learning and AdnSamples.txt, an
[include file](http://thebuildingcoder.typepad.com/blog/2008/11/loading-the-building-coder-samples.html)
for
[RvtSamples](http://thebuildingcoder.typepad.com/blog/2008/09/loading-sdk-sam.html)
that I use to load all my external commands, updated to the
[2010 RvtSamples format](http://thebuildingcoder.typepad.com/blog/2009/05/porting-the-building-coder-samples.html).

#### Revit Inside

By the way, if you are interested in seeing who else is using Revit, there is a
[Revit Inside](http://www.revitinside.com)
website which tells you just that.
The people running the list are always interested in new entries as well, so don't hesitate to check it out and sign up if it applies to you.