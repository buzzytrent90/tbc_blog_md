---
post_number: "1218"
title: "Is a Given Element Hidden in a View?"
slug: "element_hidden_view"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'revit-api', 'views']
source_file: "1218_element_hidden_view.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1218_element_hidden_view.html"
---

### Is a Given Element Hidden in a View?

Lately, several people have asked about how to determine the visibility of an element relative to a given view crop box.

Here is a nice little stand-alone method IsElementHiddenInView that can be plugged in to any add-in to answer that question, plus a couple of others as well, shared by
Frode Tørresdal of
[Norconsult Informasjonssystemer AS](https://www.nois.no).

For any given element e and view v, it returns true if e is hidden in v.

This is determined by performing the following checks:

- Does v have a crop box defined? Does e lies completely outside it? If not, is less that 25 % of the element bounding box contained within the view crop box?
- Is e specifically hidden in the view, by element?
- Is the category of e or one of its parent categories hidden in v?

Here is the code:

```csharp
  /// <summary>
  /// Return true if the given element e is hidden
  /// in the view v. This might be due to:
  /// - e lies outside the view crop box
  /// - e is specifically hidden in the view, by element
  /// - the category of e or one of its parent
  /// categories is hidden in v.
  /// </summary>
  bool IsElementHiddenInView(
    Element e,
    View v )
  {
    if( v.CropBoxActive )
    {
      BoundingBoxXYZ viewBox = v.CropBox;
      BoundingBoxXYZ elBox = e.get\_BoundingBox( v );

      Transform transInv = v.CropBox.Transform.Inverse;

      elBox.Max = transInv.OfPoint( elBox.Max );
      elBox.Min = transInv.OfPoint( elBox.Min );

      // The transform above might switch
      // max and min values.

      if( elBox.Min.X > elBox.Max.X )
      {
        XYZ tmpP = elBox.Min;
        elBox.Min = new XYZ( elBox.Max.X, elBox.Min.Y, 0 );
        elBox.Max = new XYZ( tmpP.X, elBox.Max.Y, 0 );
      }

      if( elBox.Min.Y > elBox.Max.Y )
      {
        XYZ tmpP = elBox.Min;
        elBox.Min = new XYZ( elBox.Min.X, elBox.Max.Y, 0 );
        elBox.Max = new XYZ( tmpP.X, elBox.Min.Y, 0 );
      }

      if( elBox.Min.X > viewBox.Max.X
        || elBox.Max.X < viewBox.Min.X
        || elBox.Min.Y > viewBox.Max.Y
        || elBox.Max.Y < viewBox.Min.Y )
      {
        return true;
      }
      else
      {
        BoundingBoxXYZ inside = new BoundingBoxXYZ();

        double x, y;

        x = elBox.Max.X;

        if( elBox.Max.X > viewBox.Max.X )
          x = viewBox.Max.X;

        y = elBox.Max.Y;

        if( elBox.Max.Y > viewBox.Max.Y )
          y = viewBox.Max.Y;

        inside.Max = new XYZ( x, y, 0 );

        x = elBox.Min.X;

        if( elBox.Min.X < viewBox.Min.X )
          x = viewBox.Min.X;

        y = elBox.Min.Y;

        if( elBox.Min.Y < viewBox.Min.Y )
          y = viewBox.Min.Y;

        inside.Min = new XYZ( x, y, 0 );

        double eBBArea = ( elBox.Max.X - elBox.Min.X )
          \* ( elBox.Max.Y - elBox.Min.Y );

        double einsideArea =
          ( inside.Max.X - inside.Min.X )
          \* ( inside.Max.Y - inside.Min.Y );

        double factor = einsideArea / eBBArea;

        if( factor < 0.25 )
          return true;
      }
    }

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

One especially noteworthy feature worth pointing out here is the element bounding box and view crop box comparison:

To compare the element bounding box with the view crop box, the inverse of the crop box transform needs to be applied.

It may swap the relative positions of the bounding box min and max values, so we need to check for that and correct it.

Many thanks to Frode for sharing this.

I added this method to
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2015.0.113.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.113.2).