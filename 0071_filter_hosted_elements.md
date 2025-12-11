---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 1.8
content_type: qa
optimization_date: '2025-12-11T11:44:13.314162'
original_url: https://thebuildingcoder.typepad.com/blog/0071_filter_hosted_elements.html
post_number: '0071'
reading_time_minutes: 1
series: filtering
slug: filter_hosted_elements
source_file: 0071_filter_hosted_elements.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- parameters
- revit-api
- walls
- windows
title: Filter for Hosted Elements
word_count: 222
---

### Filter for Hosted Elements

Today was the last day of the Revit programming training here in Barcelona.
One of the items we examined is
[yesterday's suggestion](http://thebuildingcoder.typepad.com/blog/2009/01/barcelona-questions.html#retrieve_windows_hosted_by_a_wall)
to use API filtering and the HOST\_ID\_PARAM to select the doors and windows hosted by a specific wall. Here is a code snippet that implements this and displays the result in a message box:

```csharp
ElementId id = wall.Id;
Type t = typeof( FamilyInstance );
BuiltInCategory bicd = BuiltInCategory.OST\_Doors;
BuiltInCategory bicw = BuiltInCategory.OST\_Windows;
BuiltInParameter bip = BuiltInParameter.HOST\_ID\_PARAM;

Autodesk.Revit.Creation.Filter cf = app.Create.Filter;
Filter f1 = cf.NewCategoryFilter( bicd );
Filter f2 = cf.NewCategoryFilter( bicw );
Filter f3 = cf.NewLogicOrFilter( f1, f2 );

Filter f4 = cf.NewTypeFilter( t );
Filter f5 = cf.NewLogicAndFilter( f3, f4 );

Filter f6 = cf.NewParameterFilter( bip,
  CriteriaFilterType.Equal, id );

Filter f7 = cf.NewLogicAndFilter( f5, f6 );

List<Element> hosted = new List<Element>();
doc.get\_Elements( f7, hosted );
n = hosted.Count;

string s = string.Format(
  "Wall <{0} {1}> hosts {2} door"
  + " and window element{3}{4}\n",
  wall.Name, id.Value, n,
  ( ( 1 == n ) ? "" : "s" ),
  ( ( 0 == n ) ? "." : ":" ) );

foreach( FamilyInstance fi in hosted )
{
  s += string.Format( "\n  {0} {1} {2}",
    fi.Category.Name, fi.Name, fi.Id.Value );
}
MessageBox.Show( s, "Anfitrion" );
```

In case you are wondering, 'anfitrión' means 'host' in Spanish.