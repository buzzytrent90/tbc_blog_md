---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.421536'
original_url: https://thebuildingcoder.typepad.com/blog/1158_views_displaying_elem.html
post_number: '1158'
reading_time_minutes: 10
series: views
slug: views_displaying_elem
source_file: 1158_views_displaying_elem.htm
tags:
- csharp
- elements
- filtering
- geometry
- levels
- revit-api
- rooms
- selection
- transactions
- views
title: Determine Views Displaying Given Element
word_count: 2021
---

### Determine Views Displaying Given Element

I am slowly getting back to normal working mode after the AEC Hackathon last weekend and the neat project that we worked on there, so I finally get around to publishing this post that I started working on last week.

The only remaining issue is of a physical nature: my feet are still swollen after flying to New York on Friday, sitting and working non-stop at the Hackathon from Saturday morning until Sunday afternoon, standing for a couple of hours in a pub afterwards, and then sitting in the flight back again the same evening and Monday. My veins are getting old, it seems, and need some horizontal time out now and then.

In the meantime, other topics cropped up as well, including some cases and emails I answered since my return:

- [SpatialElement Level property](#2)
- [Export to DWFx, SVG or XML](#3)
- [Add-in benchmarking](#4)
- [NoSQL and top five considerations](#5)
- [Determine views displaying given element](#6)
- [Download](#7)

#### SpatialElement Level Property

**Question:** The previous version of the API provided direct access to a Level object directly from a SpatialElement.
In the current version this has been made obsolete and replaced by only a LevelId.

The previous way allowed me to get the information about a Level from a particular SpatialElement that I was working with. Is there any work-around to get Level information from a SpatialElement or is it intended that I acquire the Level through the LevelId I can access from the SpatialElement?

**Answer:** I am not aware of any other method than what you suggest, nor can I imagine that any other method would or even could be more direct and efficient than that.

You could in fact implement this approach as an
[extension method](http://thebuildingcoder.typepad.com/blog/2010/02/getpolygon-extension-methods.html) on
the Revit 2015 API SpatialElement class, if you prefer to leave your existing code unchanged, e.g.

```csharp
  static Level Level( this SpatialElement a )
  {
    Document doc = a.Document;
    return doc.GetElement( a.LevelId );
  }
```

#### Export to DWFx, SVG or XML

I replied to a Revit API discussion thread on
[Revit plan view custom export](http://forums.autodesk.com/t5/Revit-API/Rvit-Plan-View-custom-export-DWFx-or-SVG-etc/m-p/5039904):

**Question:** I have some questions on DWFx and other graphical export from Revit:

1. How does the export to DWFx work?
2. Does the process simply export all visible elements on plan view (Geometry, Annotation, etc.), or is there a more
   complicated process such as printing preprocessing?

The reason I'm asking is that I would like to create my own plan view export, for example to SVG or some other XML based format of my own creation, with complete detail information including annotations, etc.

Do you think that would be possible?

**Answer:** Definitely yes.

There are a lot of answers to this.

**1.** How does the export to DWFx work?

The first answer is simple, and applies to most of the Revit API:

The Revit API export to DWFx works exactly the same way as the user interface functionality.

Call the Document.Export method overload taking a DWFXExportOptions argument to exports the current view or a selection of views in DWFX format.

For examples, please look at the Revit SDK and The Building Coder blog, e.g. searching for
[building coder dwf export](http://lmgtfy.com/?q=building+coder+dwf+export).

**2.** Does it simply export all visible elements on plan view?

It exports exactly what you see in the view.
You can set up the view to export what you want.
This includes annotation and also
[sections](http://thebuildingcoder.typepad.com/blog/2011/08/section-view-geometry.html).

**3.** I would like to create my own plan view export, for example to SVG or some other XML based format.

Here, you are in great luck:

You can look at my simplified
[2D BIM room editor](http://lmgtfy.com/?q=building+coder+room+editor) for
a very complete example of exporting 2D to SVG.

It lives on GitHub, in two repositories,
[roomedit](https://github.com/jeremytammik/roomedit), implementing the NoSQL CouchDB HTML and SVG viewer, and
[RoomEditorApp](https://github.com/jeremytammik/RoomEditorApp), implementing the Revit add-in, database upload and real-time BIM update.

It shows that you can easily not just export to SVG and display a simplified BIM view through server-side scripting-generated HTML, but edit the SVG live in the browser and update your BIM with the results in real-time as well.

That might well be exactly what you are looking for.

For a full 3D export of the Revit model to JSON and a live web viewer displaying both graphics and non-graphical element properties, you can look at what we achieved last weekend at the AEC Hackathon in New York:

- [Project description](https://www.hackerleague.org/hackathons/aec-technology-hackathon-2014/hacks/three-dot-js-aec-viewer-model-exporters)
- [Main project page](https://va3c.github.io)
- [Sample viewer](https://va3c.github.io/viewer)
- [va3c team](https://va3c.github.io/#team)
- [va3c exporter from Revit by Matt Mason and Jeremy](https://github.com/va3c/RvtVa3c)
- [Live blog post during the event](http://thebuildingcoder.typepad.com/blog/2014/05/aec-hackathon-from-the-midst-of-the-fray.html)

We won second prize!   :-)

#### Revit Add-in Benchmarking

**Question:** I am reworking a Revit add-in that I discussed with you last year.

Before I move further, I wondered if you would like to take a look at it?

I am having some performance issues that I'm sure are easily fixed.

I am using SQL and – for the first time – WPF, and loving both of them.

**Answer:** Before I take a look at it, have you benchmarked it in any way whatsoever?

That should immediately highlight any bottlenecks, and you might improve your performance significantly with a very few tweaks, or alternatively prove that no such tweaks exist, e.g. if the performance issue is caused by Revit and not your add-in.

Please look at the discussion on
[profiling Revit add-ins](http://thebuildingcoder.typepad.com/blog/2014/04/profiling-revit-add-ins-and-roomeditorapp-enhancements.html),
see what you can achieve using that, and then we can continue talking.

#### NoSQL and Top Five Considerations

Continuing with the question above on SQL:
I cannot say anything specific about SQL, but I do like NoSQL, 'Not only SQL':

- [nosql-database.org](http://nosql-database.org) ([Wikipedia](https://en.wikipedia.org/wiki/NoSQL))
- Next generation database paradigm
- Non-relational, distributed, open-source, tremendously scalable, huge data
- Mostly schema-free, easy replication support, simple API
- Eventually consistent, BASE (not ACID)

The
[CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem) states
that a distributed computer system based on a traditional ACID paradigm such as SQL cannot possibly simultaneously provide all three of the following guarantees:

- Consistency (all nodes see the same data at the same time)
- Availability (a guarantee that every request receives a response about whether it was successful or failed)
- Partition tolerance (the system continues to operate despite arbitrary message loss or failure of part of the system)

NoSQL relies on new distributed computing insights such as
[eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency),
enabling the switch from traditional rigid ACID concepts to BASE database paradigms:

- ACID

- Atomicity, Consistency, Isolation, Durability
- Traditional computer science
- Guarantee that database transactions are processed reliably

- BASE

- Basic Availability, Soft-state, Eventual consistency
- Not guaranteed to be in a consistent state at a given moment
- Consistency is guaranteed, eventually

Have you taken a look at that?

Here is a whitepaper on the
[top five NoSQL considerations](http://www.mongodb.com/nosql-explained),
including some comparison with SQL, provided by MongoDB and only slightly commercially tainted.

I have not done much with WPF, and nothing at all with SQL in the last years.

#### Determine Views Displaying Given Element

Several people recently asked about how to
[retrieve all views displaying a given element](http://forums.autodesk.com/t5/Revit-API/Revision-help-which-views-show-this-object/m-p/5029772).

I provided my standard answer to the question by looping over all views and using a filtered element collector passing in the view element id of each.

That primitive approach was enhanced by Colin, who replies:

> The solution as Jeremy suggested is certainly possible, BUT in a model with many views, it can take an extremely long time to run.
>
> The reason for this is, (as far as I can tell), when using a view-specific FilteredElementCollector, Revit effectively opens that view and checks whether the element is visible in the view.
>
> The result of this is that getting a list of views where an element is present takes the same amount of time as opening every view that is checked.
>
> That being said, I implemented a method FindAllViewsWhereAllElementsVisible and helper method FindAllViewsThatCanDisplayElements for this (cf. below).
>
> If you can't tell, I'm a big fan of LINQ... Let me know if you need me to explain anything that's happening here.

Jeremy replies to Colin saying:

Nice solution!

I like it!

You also seem to like extension methods, performance optimisation and the ElementMulticlassFilter.

So do I.

I therefore integrated your code into The Building Coder samples and implemented a new external command CmdViewsShowingElements to exercise it.

While I was at it, I also finally also reorganised the RvtSamples include file BcSamples.txt to generate three groups for The Building Coder samples instead of two to reduce the menu overflow. Previously, the commands were split into two groups A-M and N-Z listing over 50 entries each. The bottom ones were off the bottom of the screen and impossible to pick.
The new three groups list a bit over 30 entries each.

Here is the CmdViewsShowingElements implementation including Colin's two extension methods:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

namespace BuildingCoder
{
  /// <summary>
  /// Revit Document and IEnumerable<Element>
  /// extension methods.
  /// </summary>
  static class ExtensionMethods
  {
    /// <summary>
    /// Return an enumeration of all views in this
    /// document that can display elements at all.
    /// </summary>
    static IEnumerable<View>
      FindAllViewsThatCanDisplayElements(
        this Document doc )
    {
      ElementMulticlassFilter filter
        = new ElementMulticlassFilter(
          new List<Type> {
            typeof( View3D ),
            typeof( ViewPlan ),
            typeof( ViewSection ) } );

      return new FilteredElementCollector( doc )
        .WherePasses( filter )
        .Cast<View>()
        .Where( v => !v.IsTemplate );
    }

    /// <summary>
    /// Return all views that display
    /// any of the given elements.
    /// </summary>
    public static IEnumerable<View>
      FindAllViewsWhereAllElementsVisible(
        this IEnumerable<Element> elements )
    {
      if( null == elements )
      {
        throw new ArgumentNullException( "elements" );
      }

      //if( 0 == elements.Count )
      //{
      //  return new List<View>();
      //}

      Element e1 = elements.FirstOrDefault<Element>();

      if( null == e1 )
      {
        return new List<View>();
      }

      Document doc = e1.Document;

      IEnumerable<View> relevantViewList
        = doc.FindAllViewsThatCanDisplayElements();

      IEnumerable<ElementId> idsToCheck
        = ( from e in elements select e.Id );

      return (
        from v in relevantViewList
          let idList
            = new FilteredElementCollector( doc, v.Id )
              .WhereElementIsNotElementType()
              .ToElementIds()
          where !idsToCheck.Except( idList ).Any()
          select v );
    }
  }

  /// <summary>
  /// Determine all views displaying
  /// a given set of elements.
  /// </summary>
  [Transaction( TransactionMode.ReadOnly )]
  class CmdViewsShowingElements : IExternalCommand
  {
    public Result Execute(
      ExternalCommandData revit,
      ref string message,
      ElementSet elements )
    {
      UIApplication uiapp = revit.Application;
      UIDocument uidoc = uiapp.ActiveUIDocument;
      Document doc = uidoc.Document;

      // Retrieve pre-selected elements.

      ICollection<ElementId> ids
        = uidoc.Selection.GetElementIds();

      if( 0 == ids.Count )
      {
        message = "Please pre-select some elements "
          + "before launching this command to list "
          + "the views displaying them.";

        return Result.Failed;
      }

      // Determine views displaying them.

      IEnumerable<Element> targets
        = from id in ids select doc.GetElement( id );

      IEnumerable<View> views = targets
        .FindAllViewsWhereAllElementsVisible();

      // Report results.

      string names = string.Join( ", ",
        ( from v in views select v.Name ) );

      int nElems = targets.Count<Element>();

      int nViews = names.Count<char>(
        c => ',' == c ) + 1;

      TaskDialog dlg = new TaskDialog( string.Format(
        "{0} element{1} are visible in {2} view{3}",
        nElems, Util.PluralSuffix( nElems ),
        nViews, Util.PluralSuffix( nViews ) ) );

      dlg.MainInstruction = names;

      dlg.Show();

      return Result.Succeeded;
    }
  }
}
```

#### Download The Building Coder Samples

The complete source code, Visual Studio solution and RvtSamples include file is provided in
[The Building Coder samples GitHub repository](https://github.com/jeremytammik/the_building_coder_samples).

The version discussed above is
[release 2015.0.110.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.110.0).