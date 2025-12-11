---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:13.630413'
original_url: https://thebuildingcoder.typepad.com/blog/0255_visible_elements.html
post_number: '0255'
reading_time_minutes: 4
series: elements
slug: visible_elements
source_file: 0255_visible_elements.htm
tags:
- csharp
- elements
- filtering
- revit-api
- views
- walls
title: Visible Elements
word_count: 719
---

### Visible Elements

I am sitting here cramped up in an economy seat crossing the Atlantic Ocean as I write this.
I should urgently be practicing and preparing for my sessions at AU, but somehow it feels as if writing a blog post requires slightly less space, so here goes.
I would also like to answer some pending comments, but cannot right now due to lack of Internet access...

After revisiting the topic of
[selecting all model elements](http://thebuildingcoder.typepad.com/blog/2009/11/select-model-elements-2.html) in the last post,
there is one more important and interesting issue to take into account to determine all the **visible** model elements.
In that post, we made use of element filtering instead of an iterator over all elements and eliminated a number of unwanted element types and categories.
Once we have thus whittled away the non-model elements, there are still several possibilities to hide elements in a given view through the user interface.
Here are two main methods that can be used to hide elements in a view:

- Hide an individual element.- Hide an entire category of elements.

To determine whether an element is hidden, we need to check for both of these possibilities.
In addition, an element may belong to a subcategory.
If so, it may be hidden either because its subcategory or one of its parent categories is hidden.

The Revit SDK sample VisibilityControl demonstrates how to control visibility by category, and the topic of categories and how they are used is covered in detail in section 5.2.1 Category of the developer guide "Revit 2010 API Developer Guide.pdf".

The visibility of an element in a view can be queried using the Element.IsHidden( View ) method, the visibility of a category by the Category.Visible property.

Here is the code of a helper method IsHiddenElementOrCategory which takes all these factors into account and can be used to check whether an element 'e' is hidden in a given view 'v'.
It returns true if the given element is hidden in this view, either as an individual element, or because its category or one of its parent categories is hidden:
```csharp
static bool IsHiddenElementOrCategory(
  Element e,
  View v )
{
  bool hidden = e.IsHidden( v );

  if( !hidden )
  {
    Category cat = e.Category;
    while( null != cat && !hidden )
    {
      hidden = !cat.get\_Visible( v );
      cat = cat.Parent;
    }
  }
  return hidden;
}
```

#### Red Rock Revisited

By the way, some time has passed between starting to write and actually posting this.
I arrived safely in Las Vegas, grabbed a rental car, and went off to un-jetlag by spending the night in a sleeping bag in the desert.
Rather cold, but not freezing, this time around.
Two years ago, it was significantly below freezing and my gallon bottle of water froze solid, all the way down.
Last year, it was warmer and moister and had actually rained just before my arrival.
This year is the most pleasant to date.
Frankly, I prefer the desert to the strip.

I spent my first day here climbing in the
[Red Rock](http://en.wikipedia.org/wiki/Red_Rock_Canyon_National_Conservation_Area) climbing area
with a bunch of crazy guys from Dallas, Texas, namely Karl, Fritz, Gary, Jess and Ellis.
We did several easy routes on the Ultraman wall, including Scent of an Ultraman 5.7, Clutch Cargo 5.7 and Science Patrol 5.8.

We met up early next morning again to go the
[Black Corridor](http://www.rockclimbing.com/routes/North_America/United_States/Nevada/Red_Rock_Canyon/Second_Pullout/Black_Corridor),
which I visited in the previous two years as well.
I led a number of climbs in the morning including The Cel 50' 5.9, Bon Ez 5.9+, and two of hardest and nicest I've ever done, Heavy Hitter and Friend 40', both 5.10d.
The guys from Dallas left to return home at midday and I teamed up with Sonja & Jean-Francois from Quebec in the afternoon to climb two more routes led by Jean-Francois, Crude Street 5.9+ and Dance with a God 5.10a.

I am now sitting in the Barnes and Noble book store on West Charleston Boulevard and gratefully using the free Internet access they provide to finish off and post this.
This is the road that I keep going up and down to get to Red Rock.