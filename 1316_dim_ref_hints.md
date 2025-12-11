---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.7
content_type: qa
optimization_date: '2025-12-11T11:44:15.754148'
original_url: https://thebuildingcoder.typepad.com/blog/1316_dim_ref_hints.html
post_number: '1316'
reading_time_minutes: 5
series: general
slug: dim_ref_hints
source_file: 1316_dim_ref_hints.htm
tags:
- csharp
- elements
- family
- geometry
- levels
- references
- revit-api
- selection
- views
- walls
- windows
title: How to Retrieve Dimensioning References
word_count: 1063
---

### How to Retrieve Dimensioning References

Several cases recently came up asking how to obtain references to programmatically create dimensioning elements.

These hints expand on the recently discussed topic of creating
[dimensioning between family instance insertion points](http://thebuildingcoder.typepad.com/blog/2014/11/picking-pairs-and-dimensioning-family-instance-origin.html).

They were raised by the following queries on how to retrieve suitable references for dimensioning:

- [Dimensioning to family instance centre reference planes](#2)
- [Dimensioning between a grid and an edge of a face](#3)
- [Dimensioning to the wall opening wrapping location](#4)

Before getting to that, I'll just add that I returned safe and sound from the
[Dubai hackathon](http://thebuildingcoder.typepad.com/blog/2015/05/cloning-a-solid-angelhack-3d-web-fest-and-dubai.html) that I mentioned last Friday.

I reported on the event during the weekend, discussing
[hackathon preparation](http://the3dwebcoder.typepad.com/blog/2015/05/dubai-hackathon-preparation-and-viewer-workshop.html#6) in general, the
[View and Data API workshop](http://the3dwebcoder.typepad.com/blog/2015/05/dubai-hackathon-preparation-and-viewer-workshop.html#7) and the
[hackathon projects presented](http://the3dwebcoder.typepad.com/blog/2015/05/dubai-hackathon-project-presentation-notes.html).

Back to the Revit API and the topic at hand:

#### Dimensioning to Family Instance Centre Reference Planes

**Question:**
I want to get the reference planes in a family instance for dimension creation with the API.

For example, use the API to create a dimension between the centre references of two windows.

The following test code, for example, does not find the reference planes I need:

```csharp
  public void ListFamilyGeometry( UIDocument uidoc )
  {
    Document doc = uidoc.Document;

    Reference r;

    try
    {
      r = uidoc.Selection.PickObject(
        ObjectType.Element );
    }
    catch( Autodesk.Revit.Exceptions
      .OperationCanceledException )
    {
      return;
    }

    FamilyInstance fi = doc.GetElement( r )
      as FamilyInstance;

    if( fi == null )
    {
      return;
    }

    Transform transform = fi.GetTransform();

    string data = string.Empty;

    Options options = new Options();
    options.IncludeNonVisibleObjects = true;

    foreach( GeometryObject go in
      fi.get\_Geometry( options ) )
    {
      data += go.GetType().ToString()
        + Environment.NewLine;

      if( go is GeometryInstance )
      {
        GeometryInstance gi = go as GeometryInstance;

        foreach( GeometryObject goSymbol in
          gi.GetSymbolGeometry() )
        {
          data += " - " + goSymbol.GetType().ToString()
            + Environment.NewLine;

          if( goSymbol is Line )
          {
            Line line = goSymbol as Line;

            makeLine( doc, transform.OfPoint(
              line.GetEndPoint( 0 ) ),
              transform.OfPoint( line.GetEndPoint( 1 ) ) );
          }
        }
      }
    }
    TaskDialog.Show( "data", data );
  }
```

**Answer:**
I assume you have read about [dimensioning between family instance origins](http://thebuildingcoder.typepad.com/blog/2014/11/picking-pairs-and-dimensioning-family-instance-origin.html).

If your window symbols have their insertion points at the same location as the centre reference, you could use that approach.

To address your real requirement, though, you need to retrieve the centre references directly.

Here are some important aspects that you need to keep in mind to retrieve the references to create dimensioning between the centre references of two family instances, e.g. windows:

- IncludeNonVisibleGeometry is required to be set, just as you have done in your sample code snippet.
- The geometry must be extracted from a view where the reference would be useful. In the example below, no View is passed to the geometry options. In that case, the model level geometry is extracted, which lacks view specific references like reference planes orthogonal to the view.
- Once you pass in a suitable view and obtain these extra references, there is nothing to identify them. You have to analyse the geometry to figure out which one is the 'centre' one. For 2D views, reference planes may be returned as lines (Curve instances) instead of planes.

These aspects have been mentioned in the past; still, I hope it helps to spell them out explicitly again here and now.

#### Dimensioning Between a Grid and an Edge of a Face

The following query was raised by Samer Habib in a
[comment](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html?cid=6a00e553e16897883301b7c788f437970b#comment-6a00e553e16897883301b7c788f437970b) on
[What's New in the Revit 2016 API](http://thebuildingcoder.typepad.com/blog/2015/04/whats-new-in-the-revit-2016-api.html):

**Question:**
I am trying to create a new dimension between a grid and an edge of a face.

I get the reference of grid by grid.Curve.reference and I get the reference of the face edge.

However, in the NewDimension method the references argument requires an array of geometric references to which the dimension is to be bound.

The grid does not seem to have any such geometric reference.

Therefore, when I call the NewDimension method, it throws an 'Invalid number of references' error.

How can I solve this, please?

**Answer:**
The answer is exactly the same as above:

Please note the description of [dimensioning between instance insertion points](http://thebuildingcoder.typepad.com/blog/2014/11/picking-pairs-and-dimensioning-family-instance-origin.html), plus these additional important aspects:

- Set IncludeNonVisibleGeometry when requesting the element geometry.
- The geometry must be extracted from a view where the reference would be useful. In the example below, no View is passed to the geometry options. In that case, the model level geometry is extracted, which lacks view specific references like reference planes orthogonal to the view.
- Once you pass in a suitable view and obtain these extra references, there is nothing to identify them. You have to analyse the geometry to figure out which one is the one you need. For instance, for 2D views, reference planes may be returned as lines (Curve instances) instead of planes.

**Response:**
Yes!

I now set the options like this:

```csharp
  Options goption = new Options();
  goption.ComputeReferences = true;
  goption.IncludeNonVisibleObjects = true;
  goption.View = doc.ActiveView;
```

That returns the grid geometry and the problem is solved.

#### Dimensioning to the Wall Opening Wrapping Location

Here is another query on dimensioning to a specific location that does not directly provide any built-in references of its own:

**Question:**
Can the API be used to create a dimension to the wrapping location of an insert in a wall?

Here is an example of the desired dimension:

![Dimension to wall opening wall wrap location](img/dimension_to_wall_opening_wall_wrap_location.png)

If it is not possible to get a reference to this location, can the location be found so a detail line can be created which could be used for the dimension?

**Answer:**
Yes, this location can be found as described in the discussion on [retrieving detailed wall layer geometry](http://thebuildingcoder.typepad.com/blog/2011/10/retrieving-detailed-wall-layer-geometry.html).