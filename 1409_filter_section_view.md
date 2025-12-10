---
post_number: "1409"
title: "Filter Section View"
slug: "filter_section_view"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'python', 'references', 'revit-api', 'sheets', 'views', 'walls']
source_file: "1409_filter_section_view.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1409_filter_section_view.html"
---

### API, SDK and View Section Box Element Intersection
Trying to keep track of the overwhelming information flow provided by the numerous discussions on
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) is impossible.
Here are two little titbits from the past few days:
- [Basic Revit API and SDK Access, Online and Offline](#2)
- [Filtering for Elements Intersecting a View Section Box](#3)
#### Basic Revit API and SDK Access, Online and Offline
Here is a very basic question, raised by Miguel Rus in the Revit API discussion forum thread
on [Revit API online](http://forums.autodesk.com/t5/revit-api/revit-api-online/m-p/6061199) that
I reproduce here for the sake of clarity and future reference.
It belongs to the category of things to read and understand even before getting started :-)
\*\*Question:\*\* I am trying to find an the online API for Revit 2016 but I can't seem to finds it.
Could someone point me in the right direction.
The offline CHM is not of any help as the firewall at my office is blocking the connection to the Internet.
\*\*Answer:\*\* The Revit API, the application developer interface, is always included with Revit itself.
As far as I know, the end user product cannot run without the API installed. The API is provided by certain .NET assemblies located in the same folder as Revit.exe itself, such as `RevitAPI.dll` and `RevitAPIUI.dll`.
I guess what you mean is the Revit SDK, the software developers kit.
It contains important and helpful API documentation and samples, such as the Revit API help file `RevitAPI.chm` that you refer to.
The SDK is installed by running an executable file, `RevitSDK.exe`.
You can install the SDK from various places. It is included under 'Tools' in the standard Revit product installation.
It is also available from the main entry point for information on and getting started with the Revit API,
the [Revit Developer Centre](http://www.autodesk.com/developrevit).
The Building Coder provides more hints
on [getting started with the Revit API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#2).
If you are asking specifically about Revit API help online, then The Building Coder discussion
about [Revit API help online](http://thebuildingcoder.typepad.com/blog/2014/01/revit-api-help-online-and-hiking-on-la-palma.html) might
possibly help.
:-)
Unfortunately, the version pointed to there is for Revit 2014 and has not been updated: [www.revitapisearch.com](http://www.revitapisearch.com).
I would recommend simply using the CHM provided by the SDK; you can rely on that, always, store it locally, always keep it at hand, whether online or off...
#### Filtering for Elements Intersecting a View Section Box
Here is a useful and important piece of information on determining all elements intersecting a view's section box, shared by Jan Tepelmann
at [Inreal Technologies](https://www.inreal-tech.com) in the Revit API discussion forum thread
on [BoundingBoxIntersectsFilter not working](http://forums.autodesk.com/t5/revit-api/boundingboxintersectsfilter-not-working/td-p/5912235):
\*\*Question:\*\* I'm a programmer at Inreal Technologies and were developing Enscape 3D, a rendering plugin for Revit.
Currently we are working on speeding up the switching between two 3D views. Right now we just recreate the scene to render but this is obviously very slow.
So I came up with the following code to find out which elements were added, removed and modified when you switch from one view (the start view) to another view (the target view). The idea behind the code is, that we want to know all elements which got cut by the start view's section box or will get cut by the target view section box. For those elements the geometry has to be updated:

```
  public static DiffUpdateElementIds Calculate(
    View3D startView, View3D targetView )
  {
    var diffElements = new DiffUpdateElementIds();

    //... here added and removed elements are calculated
    // (with exclude filter). This works just fine

    if( startView.IsSectionBoxActive
      || targetView.IsSectionBoxActive )
    {
      ElementFilter intersectFilterStart = null;
      ElementFilter intersectFilterTarget = null;
      if( startView.IsSectionBoxActive )
      {
        Logger.info( "start View has SectionBox" );
        intersectFilterStart
          = new BoundingBoxIntersectsFilter(
            new Outline( startView.GetSectionBox().Min,
              startView.GetSectionBox().Max ) );
      }
      if( targetView.IsSectionBoxActive )
      {
        Logger.info( "target View has SectionBox" );
        intersectFilterTarget
          = new BoundingBoxIntersectsFilter(
            new Outline( targetView.GetSectionBox().Min,
              targetView.GetSectionBox().Max ) );
      }

      ElementFilter intersectFilter;
      if( intersectFilterStart != null
        && intersectFilterTarget == null )
      {
        intersectFilter = intersectFilterStart;
      }
      else if( intersectFilterStart == null
        && intersectFilterTarget != null )
      {
        intersectFilter = intersectFilterTarget;
      }
      else
      {
        intersectFilter = new LogicalOrFilter(
          intersectFilterStart, intersectFilterTarget );
      }

      var cutElementsStartView
        = new FilteredElementCollector(
          startView.Document, startView.Id )
            .WherePasses( intersectFilter );

      var cutElementsTargetView
        = new FilteredElementCollector(
          targetView.Document, targetView.Id )
            .WherePasses( intersectFilter );

      diffElements.ModifiedElements
        = cutElementsStartView.UnionWith(
          cutElementsTargetView ).ToElementIds();
    }
    return diffElements;
  }
```

The problem now is that it misses a ceiling in the Advanced Sample Project when I switch from the default 3D view to "03 - Floor Public - Day Rendering" view, cf. this screen snapshot:
![Ceiling not cut](img/ceiling_not_cut.png)
What am I missing? I have to say that I'm relatively new to the filtering API so any help is appreciated.
\*\*Answer:\*\* I could solve the problem now by myself.
Maybe it’s useful for someone else who has the same problem:

```
  if( startView.IsSectionBoxActive )
  {
    Transform t = startView.GetSectionBox().Transform;

    Outline o = new Outline(
      t.OfPoint( startView.GetSectionBox().Min ),
      t.OfPoint( startView.GetSectionBox().Max ) );

    intersectFilterStart
      = new BoundingBoxIntersectsFilter( o );
  }
  if( targetView.IsSectionBoxActive )
  {
    Transform t = targetView.GetSectionBox().Transform;

    Outline o = new Outline(
      t.OfPoint( targetView.GetSectionBox().Min ),
      t.OfPoint( targetView.GetSectionBox().Max ) );

    intersectFilterTarget
      = new BoundingBoxIntersectsFilter( o );
  }
```

The code I was missing is to apply the view section box transform. I was just missing that the section box can have a transformation. I thought the box is always defined in world space, but that's not true. Sometimes it is not and then the old code did not work. So hopefully this helps someone who has the same issue.
Now everything is working and the switch between two 3D views is ***much*** faster then before.
Great! :-)
Many thanks to Jan for sharing this!