---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.6
content_type: news
optimization_date: '2025-12-11T11:44:13.628112'
original_url: https://thebuildingcoder.typepad.com/blog/0254_select_model_elements_2.html
post_number: '0254'
reading_time_minutes: 4
series: elements
slug: select_model_elements_2
source_file: 0254_select_model_elements_2.htm
tags:
- csharp
- elements
- family
- filtering
- geometry
- levels
- references
- revit-api
- rooms
- selection
- sheets
- views
- walls
title: Select Model Elements 2
word_count: 765
---

### Select Model Elements 2

We already had several stabs at the topic of
[selecting all model elements](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html),
i.e. all elements in a Revit BIM which contribute to the building geometry.
Their main characteristic is that they have a non-null geometry which is visible on the graphics screen and really contributes to the physical building model.
Unfortunately, some elements which are not part of the building itself also have valid geometry and need to be eliminated.

The latest version of the code that we presented still made use of a full iteration over all document elements, i.e. something like this:
```csharp
  Application app = commandData.Application;
  Document doc = app.ActiveDocument;

  ElementIterator it = doc.Elements;

  while( it.MoveNext() )
  {
    Element e = it.Current as Element;
    // . . .
  }
```

As we all know by now, this kind of full iteration over all document elements is unnecessarily time consuming and seldom useful, because an application is almost always only interested in a small subset of them.
We will convert the model element selection to use element filtering, similarly to the
[conversion to use filters](http://thebuildingcoder.typepad.com/blog/2009/02/converting-to-filters.html) we
described for the Revit SDK FrameBuilder sample.

We can add both a type filter to eliminate a list of classes that we are not interested in, and also a built-in category filter to eliminate unwanted categories.
Here is the list of unwanted types, i.e. classes of elements which are not model elements:
```csharp
static Type[] \_types\_to\_skip = new Type[] {
  typeof( BasePoint ),
  typeof( DetailLine ),
  typeof( Dimension ),
  typeof( Family ),
  typeof( FamilyBase ),
  typeof( FillPattern ),
  typeof( gbXMLParamElem ),
  typeof( GraphicsStyle ),
  typeof( Level ),
  typeof( LinePattern ),
  typeof( MaterialOther ),
  typeof( ModelLine ),
  typeof( Phase ),
  typeof( PrintSetting ),
  typeof( ProjectInfo ),
  typeof( ProjectUnit ),
  typeof( ReferencePlane ),
  typeof( Room ),
  typeof( RoomTag ),
  typeof( Sketch ),
  typeof( SketchPlane ),
  typeof( Symbol ),
  typeof( TextNote ),
  typeof( ViewDrafting ),
  typeof( ViewPlan ),
  typeof( ViewSheet ),
  typeof( WallType ),
};
```

If you run into some additional unwanted element in your model that is not listed here, you can easily add it.
You might want to let us know as well by submitting a comment on it.

We define the following helper method to more succinctly create a type filter which eliminates all of these classes.
Given an existing type filter 'f', it adds another type 't' to it and returns the new expanded filter:
```csharp
static Filter OrType(
  Filter f,
  Type t,
  Autodesk.Revit.Creation.Filter cf )
{
  Filter f2 = cf.NewTypeFilter( t );
  return cf.NewLogicOrFilter( f, f2 );
}
```

We can make use of it to implement the first step of our external command Execute method like this:
```csharp
Application app = commandData.Application;
Document doc = app.ActiveDocument;

Autodesk.Revit.Creation.Filter cf
  = app.Create.Filter;

List<Element> els = new List<Element>();
Filter f = cf.NewTypeFilter( typeof( Symbol ) );
foreach( Type t in \_types\_to\_skip )
{
  f = OrType( f, t, cf );
}
f = cf.NewLogicNotFilter( f );
int n = doc.get\_Elements( f, els );
```

In addition to the unwanted types, we know some built-in categories whose elements also do not contribute to the building geometry and should be eliminated, for instance:
```csharp
static int[] \_bics\_to\_skip = new int[] {
  ( int ) BuiltInCategory.OST\_PreviewLegendComponents,
  ( int ) BuiltInCategory.OST\_IOSSketchGrid,
  ( int ) BuiltInCategory.OST\_Cameras,
};
```

We have converted them to integer values to make use of the
[language independent category comparison](http://thebuildingcoder.typepad.com/blog/2009/06/category-comparison-and-model-element-selection-revisited.html).
The following helper method encapsulates the code to determine whether a given built-in category integer value is included in this list and the associated element thus needs to be skipped.
It makes use of the generic Array.Exists method:
```csharp
static bool SkipThisBic( int bic )
{
  return Array.Exists<int>(
    \_bics\_to\_skip,
    delegate( int i )
    {
      return i == bic;
    } );
}
```

With these preparations in place, we can iterate over and list the model elements like this in the rest of the implementation of the external command Execute method:
```csharp
string s = string.Empty;
n = 0;

Geo.Options opt = app.Create.NewGeometryOptions();

foreach( Element e in els )
{
  if( (null != e.Category)
    && !SkipThisBic( e.Category.Id.Value )
    && (null != e.get\_Geometry( opt )) )
  {
    ++n;
    s += string.Format(
      "\r\n  Category={0}; Name={1}; Id={2}",
      e.Category.Name, e.Name, e.Id.Value );
  }
}

s = string.Format(
  "Project contains {0} model element{1}{2}",
  n,
  ( 1 == n ? "" : "s" ),
  ( 0 == n ? "." : ":" ) )
+ s;

LabUtils.InfoMsg( s );

return CmdResult.Failed;
```

Here is today's snapshot of the updated version of the
[Revit API introduction labs](zip/rac_labs_2009-11-24.zip),
including the version of Lab 2-2 to select all model elements as described above.