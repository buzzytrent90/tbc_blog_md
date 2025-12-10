---
post_number: "1379"
title: "Custom Export 2d"
slug: "custom_export_2d"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'python', 'revit-api', 'rooms', 'sheets', 'views']
source_file: "1379_custom_export_2d.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1379_custom_export_2d.html"
---

### DevDay@AU and Using a Custom Exporter for 2D
Several developers recently asked how to export 2D shapes through a [custom exporter](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1).
I recently discussed this issue with Arnošt Löbel and below present a summary and combination of several different conversation threads on the topic of [using a custom exporter for 2D](#2).
First, let me mention that I arrived safe and sound at Autodesk University.
I travelled on my birthday last week, so I was upgraded to fly in style – in a sledge!
![Jeremy in sledge class](/p/2015/2015-11-26_gatwick/jeremy_xmas_sledge_400x550.jpg)
At least I got to have an extra long birthday of 32 hours instead of 24.
I also spent a couple of cold nights out in the Nevada desert, mainly contemplating the views and sleeping long hours:
[![Sleeping in the Desert](https://farm1.staticflickr.com/619/23410878355_a3024f4b90_n.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157659495325804 "Sleeping in the Desert")
Anyway, Autodesk University starts tomorrow, and the very well attended DevDay conference took place today:
[![DevDay@AU](https://farm6.staticflickr.com/5685/22808116483_6e312a0790_n.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157661173495609 "DevDay@AU")
And now back to the Revit API:
#### Using a Custom Exporter for 2D
\*\*Question:\*\* I'm trying to obtain the 2D representation of a given element in my model. I'm using a CustomExporter object to get all the visible elements in the current 3D view. I've been reading several posts and found in one of them, on the [extrusion analyser and plan view boundaries](http://thebuildingcoder.typepad.com/blog/2013/04/extrusion-analyser-and-plan-view-boundaries.html#6), that I should do something like "switch to a 2D plan view instead, and ask for the view-specific family instance representation". Now I'm stuck in how to get the correct 2D plan view if I actually have several in my model. Could you point me in the right direction?
For instance, to make it really short: Is there any way to export annotation symbols?
\*\*Answer:\*\* Take a look at the [custom exporter](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1).
In Revit 2015 it was 3D only.
In Revit 2016 it also supports 2D (1).
Take a look at my [room editor add-in](https://github.com/jeremytammik/RoomEditorApp).
There are recordings, AU class material and lots of blog posts on it.
They include SVG export (2).
Combine (1) and (2).
\*\*Response:\*\* Do you mean using the CustomExporter class? The problem is that all Export methods need a 3D view as input but I cannot get any for annotations. As far as I know, the Revit 2016 CustomExporter supports 2D output but still requires a 3D view as a starting point.
Can I really grab 2D geometry from a 2D view via custom exporter?
\*\*Answer:\*\* You are totally correct – Custom Exporter can still export 3D views only. If a 2D view is given to it, an exception will be thrown.
However, you are also correct that since Revit 2016 the custom exporter can also export text, which is a 2D entity. It also can export model lines – they are 3D entities, but they weren't exported in previous versions because they do not render.
\*\*Question:\*\* Are there any plans or even wishes or vague thoughts on removing this limitation some time in the future?
Is there any way we could work around this limitation as it currently stands?
Can we create a 3D view that looks exactly like a 2D one?
Can we somehow view the annotation symbols in a 3D view?
\*\*Answer:\*\* No, sorry:
- There are no immediate plans to support 2D custom export.
- I do not think you can create a 3D view that will look exactly like a 2D, but it greatly depends on the view and what's in it.
- You will get text output when exporting a 3D view. Try out and see what kind of annotation is included. Presumably, if you can see it in the 3D view it should make its way to the export context as well.
\*\*Response:\*\* Thanks for your reply. I found The Building Coder discussion on [views displaying a given element, SVG and NoSQL](http://thebuildingcoder.typepad.com/blog/2014/05/views-displaying-given-element-svg-and-nosql.html) that was very helpful and ended up using a similar approach. In the post you use an ElementMulticlassFilter to filter View3D, ViewPlan and ViewSection that can display a certain list of elements. In my case I just need all the plan views in the model. I iterate over that list until I find one that displays the element I need. Once I find the view I send it to an Options object and then send that object to the element get_Geometry method to get the correct 2D representation of my element.
Here is the code showing how this can be done:

```
internal class YBExporteContext : IExportContext
{
  private Document _host_document;
  private IEnumerable<View> _2D_views_that_can_display_elements;

  public YBExporteContext(
    Document document,
    View activeView )
  {
    this._host_document = document;
    this._2D_views_that_can_display_elements
      = YbUtil.FindAllViewsThatCanDisplayElements(
        document );
  }

  /*
    * Lot of code here implementing the
    * "IExportContext" interface...
    */

  private GeometryElement _get2DRepresentation(
    Element element )
  {
    View view = this._get2DViewForElement( element );
    if( view == null )
      return null;

    Options options = new Options();
    options.View = view;
    return element.get_Geometry( options );
  }

  ///
  /// Gets any 2D view where the element is displayed
  ///
  ///
  /// A 2D view where the element is displayed
  private View _get2DViewForElement( Element element )
  {
    FilteredElementCollector collector;
    ICollection<ElementId> elements_in_view;

    foreach( View view in
      this._2D_views_that_can_display_elements )
    {
      collector = new FilteredElementCollector(
        this._host_document, view.Id )
          .WhereElementIsNotElementType();

      elements_in_view = collector.ToElementIds();

      if( elements_in_view.Contains( element.Id ) )
        return view;
    }

    return null;
  }
}

public static class YbUtil
{
  public static IEnumerable<View>
    FindAllViewsThatCanDisplayElements(
      Document doc )
  {
    ElementMulticlassFilter filter
      = new ElementMulticlassFilter( new List<Type>
        { typeof( ViewPlan ) } );

    return new FilteredElementCollector( doc )
      .WherePasses( filter )
      .Cast<View>()
      .Where( v => !v.IsTemplate && v.CanBePrinted );
  }
}
```

\*\*Answer:\*\* Great solution! Thank you for sharing it.
For my room editor, I create a simplified 2D view from the 3D geometry using the extrusion analyser, and that also works well.
There are lots of different approaches, and you will have to try out for yourself which one works best for you.
Please let us know what you end up using.
Good luck and have fun!