---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: news
optimization_date: '2025-12-11T11:44:15.414313'
original_url: https://thebuildingcoder.typepad.com/blog/1153_bipchecker_2015_git.html
post_number: '1153'
reading_time_minutes: 3
series: general
slug: bipchecker_2015_git
source_file: 1153_bipchecker_2015_git.htm
tags:
- csharp
- elements
- parameters
- references
- revit-api
- selection
- views
title: Project Solon and BipChecker for Revit 2015 on GitHub
word_count: 531
---

### Project Solon and BipChecker for Revit 2015 on GitHub

One of the handful of Revit database exploration tools that I use on a regular basis is
[BipChecker](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.34),
the Revit built-in parameter checker, so that is a pretty obvious candidate for migration to Revit 2015... [see below](#3).

Before I get to that, let me mention some new Revit energy analysis functionality that you might want to check out:

#### Simple Interactive Energy Analysis in Revit with Project Solon

[Project Solon beta arrives in Revit](http://autodesk.typepad.com/bpa/2014/04/project-solon-beta-arrives-in-revit-get-more-interactive-energy-analysis-results-create-your-own-and-publish-them-to-revit.html),
providing more interactive energy analysis results, the ability to create your own and publish them to Revit.

[Autodesk Green Building Studio](https://gbs.autodesk.com/GBS) (GBS) has released the Beta version of Analysis Application Dashboard in
[Project Solon](http://help.autodesk.com/view/BUILDING_PERFORMANCE_ANALYSIS/ENU/?guid=GUID-80C2B132-8472-4C0B-A711-699AB22A4B90):
You can now author your Revit model once and deliver energy analysis results to your software application, web, and mobile device. Project Solon delivers responsive results designs for your energy analysis, with customized dynamic charts of energy results.

Read all about it on the
[Building Perfomance Analysis blog](http://autodesk.typepad.com/bpa/2014/04/project-solon-beta-arrives-in-revit-get-more-interactive-energy-analysis-results-create-your-own-and-publish-them-to-revit.html).

#### BipChecker GitHub Repository and Migration to Revit 2015

Back to the BipChecker migration: while I was at it, I also created the
[BipChecker GitHub repository](https://github.com/jeremytammik/BipChecker) to
host it, and the initial
[release 2014.0.0.3](https://github.com/jeremytammik/BipChecker/releases/tag/2014.0.0.3) based
on the existing Revit 2014 code.

Once again, the migration was very straightforward: change the .NET framework and update the Revit API assembly reference paths.

The initial flat migration is available as
[release 2015.0.0.3](https://github.com/jeremytammik/BipChecker/releases/tag/2015.0.0.3) and generates just
[two compilation warnings](zip/BipChecker_2015a.txt) concerning
the use of the UIDocument.Selection.Elements property:

- Warning CS0618: 'Autodesk.Revit.UI.Selection.Selection.Elements' is obsolete: 'This property is deprecated in Revit 2015. Use GetElementIds() and SetElementIds instead.'

Here is one example of the Revit 2014 code causing the warning, from the GetSingleSelectedElementOrPrompt method:

```csharp
  public static Element
    GetSingleSelectedElementOrPrompt(
      UIDocument uidoc )
  {
    Element e = null;
    ElementSet ss = uidoc.Selection.Elements;

    if( 1 == ss.Size )
    {
      ElementSetIterator iter = ss.ForwardIterator();
      iter.MoveNext();
      e = iter.Current as Element;
    }

    // . . .
```

Moving this to Revit 2015 to eliminate the warning is straightforward – and the result looks better   :-)

```csharp
    ICollection<ElementId> ids
      = uidoc.Selection.GetElementIds();

    if( 1 == ids.Count )
    {
      foreach( ElementId id in ids )
      {
        e = uidoc.Document.GetElement( id );
      }
    }
```

I fixed that and published the result as
[BipChecker release 2015.0.0.4](https://github.com/jeremytammik/BipChecker/releases/tag/2015.0.0.4),
which now compiles with zero wrnings.

Of course, you should normally always go for the most up-to-date master branch, though, to pick up possible later enhancements as well.