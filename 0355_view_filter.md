---
post_number: "0355"
title: "Filter for Views and the IsTemplate Predicate"
slug: "view_filter"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'levels', 'revit-api', 'sheets', 'views']
source_file: "0355_view_filter.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0355_view_filter.html"
---

### Filter for Views and the IsTemplate Predicate

Several people encountered issues when retrieving views in Revit 2011.
The reason is that the most direct approach to filtering for view elements using the new
[element filtering](http://thebuildingcoder.typepad.com/blog/2010/04/collector-benchmark.html) also
returns view templates, which were not accessible in the Revit 2010 API.
Here is the initial issue:

**Question:** I tried to extract all PlanView elements from the model and found something weird: besides the expected and obvious plan views such as Level 1, Level 2 and Site, some other unexpected ones also showed up in the collector results, such as Architectural Plan, Structural Framing Plan, Architectural Reflected Ceiling Plan, and Site Plan. I verified that this behaviour is not model specific, and that even an empty project returns all these unexpected plan views.

Why is this happening? What can I do to recognise and eliminate these unexpected and invisible plan views?

**Answer:** The unexpected plan view elements that you are seeing are the View Templates.
They used to be excluded from the API iteration, for a reason that is lost to antiquity.
The new iteration didnt know about this.
It also turns out that it does not really make sense to exclude them.

We have added a new predicate to the API, the View.IsTemplate property, which allows you to filter them out.

Here is a related question with some sample code:

**Question:** I am having difficulties with the new API in relation to retrieving unfiltered elements in a view.
From a preselected list of elements I am trying to find out in which views these elements occur.

In 2010 I used to get the elements in a view by
```csharp
List<Element> ViewElements
  = view.Elements.OfType<Element>().ToList();
```

It could not be simpler than this :-)

However, the Elements collection is no longer available, and the only way to access elements is by creating a FilteredElementCollector.

I apply the WhereElementIsNotElementType filter and then use the following code to get a list of views in which a list of elements are applicable:
```csharp
public static List<View> GetElementViews(
  List<Element> a,
  List<View> views )
{
  List<View> returnViews = new List<View>();
  foreach( View view in views )
  {
    FilteredElementCollector coll
      = new FilteredElementCollector(
        view.Document, view.Id );

    coll = coll.WhereElementIsNotElementType();

    List<Element> elementList = coll.ToList();

    foreach( Element e1 in a )
    {
      Element e2 = elementList.Where(
        x => x.Id == e1.Id )
        .FirstOrDefault();

      if( e2 != null
        && null == returnViews.Where(
          x => x.Id == view.Id ).FirstOrDefault() )
      {
        returnViews.Add( view );
      }
    }
  }
  return returnViews;
}
```

This seems to work for quite a few views, but at a certain moment it causes an exception.

**Answer:** The reason is that the following element collection returns some additional view elements than what are not visible in the user interface:
```csharp
  FilteredElementCollector coll
    = new FilteredElementCollector( doc );

  coll.OfClass( typeof( View ) );
```

The LINQ query that you apply above fails on these additional views, causing an error message:

![Invalid view id](img/invalid_view_id.png)

Some of these additional views are template views.
In theory, one way to recognise them would be to check for their crop box property like this:
```csharp
public static void GetViewsAndDrawingSheets(
  Document doc,
  List<View> views,
  List<ViewSheet> viewSheets )
{
  FilteredElementCollector coll
    = new FilteredElementCollector( doc );

  coll.OfClass( typeof( View ) );

  foreach( Element e in coll )
  {
    if( e is View )
    {
      View view = e as View;
      if( null != view.CropBox )
        views.Add( view );
    }
    else if( e is ViewSheet )
    {
      viewSheets.Add( e as ViewSheet );
    }
  }
}
```

However, a more direct and efficient way to filter them out is by using the new Revit 2011 View.IsTemplate predicate that we mentioned above:
```csharp
  foreach( Element e in coll )
  {
    if( e is View )
    {
      View view = e as View;
      if( !view.IsTemplate )
        views.Add( view );
    }
    else if( e is ViewSheet )
    {
      viewSheets.Add( e as ViewSheet );
    }
  }
```

And now for something slightly off-topic which may be of interest to some of you:

#### UK PhD Bursaries

Claudio Benghi of the
[Northumbria University](http://www.northumbria.ac.uk) in
Newcastle UK sent me a note on two PhD posts for construction-aware coders.
They come with paid fees and 13.000 GBP yearly tax-free salaries.
The deadline is next Monday, May 3rd.
The two topics are:

- SEEmail: Spatially Enhanced Email for the Construction Industry- XPIDITE: eXchange Protocols In DesIgn Teams Environments

For further information, please contact Claudio or Stephen Lockley, Professor in Building Modelling, at the
School of the Built Environment,
[Northumbria University](http://www.northumbria.ac.uk),
tel. +44 (0)191 227 4819.