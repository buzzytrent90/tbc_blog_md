---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:15.138442'
original_url: https://thebuildingcoder.typepad.com/blog/1021_multi_reference_annot.html
post_number: '1021'
reading_time_minutes: 2
series: general
slug: multi_reference_annot
source_file: 1021_multi_reference_annot.htm
tags:
- csharp
- elements
- references
- revit-api
- transactions
- views
title: MultiReferenceAnnotation Example
word_count: 482
---

### MultiReferenceAnnotation Example

Miguel Angel Alanis asked on the Autodesk Revit API discussion forum for an
[example of making use of the new MultiReferenceAnnotation](http://forums.autodesk.com/t5/Autodesk-Revit-API/An-Example-for-using-MultiReferenceAnnotation/td-p/4412817).

I put together this blog post to make sure the solution is easily found.

First, a note on something more imminent:

#### Portathon Day 1

Today is the first day of the Portathon!

- [Earn $100 Submitting an Autodesk Exchange App](http://thebuildingcoder.typepad.com/blog/2013/07/earn-100-submitting-an-autodesk-exchange-app.html)
- [Exchange App Videos](http://thebuildingcoder.typepad.com/blog/2013/08/exchange-app-videos-and-devtv-youtube-channel.html)
- [Portathon reminder and video](http://thebuildingcoder.typepad.com/blog/2013/08/adn-training-material-on-github-and-portathon-reminder.html#2)
- [Cloud and AppStore usage growth](http://thebuildingcoder.typepad.com/blog/2013/09/cloud-and-appstore-usage-grows-portathon-reminder.html)

Wish us all luck and much success, please.

Looking forward to seeing you there!

#### MultiReferenceAnnotation Creation Sample Code

Back to the main topic:

**Question:** Is it possible to get an example to create a multi-rebar annotation?

I searched the Revit SDK and found I have to use the MultiReferenceAnnotation class, but I can't build an example with the current information.

I use the Autodesk.Revit.Creation.Document.Create.NewTag method to create a normal rebar tag, and I notice the independent tag has a property named MultiReferenceAnnotationId.

**Answer:** Please note the MultiReferenceAnnotation.Create method taking a MultiReferenceAnnotationOptions argument.

The minimum options are probably the type and ElementsToDimension.

Here is an excerpt from an internal test suite.
It refers to a couple of hardcoded element ids.
Use the documentation to figure out their use:

```csharp
  var view = GetElement<View>( 124186 );

  var rebarList = GetElements<Element>(
    doc, new[] { 280427 } ).ToList();

  Assert.IsTrue( rebarList.Count > 0,
    "There are no rebars in the document!" );

  IList<ElementId> elementIds
    = new List<ElementId>();

  foreach( Element rebar in rebarList )
  {
    elementIds.Add( rebar.Id );
  }

  MultiReferenceAnnotationType type
    = GetElement<MultiReferenceAnnotationType>(
      doc, 260544 );

  Assert.IsNotNull( type,
    "the MultiReferenceAnnotationType does not exist!" );

  MultiReferenceAnnotationOptions options
    = new MultiReferenceAnnotationOptions( type );

  options.TagHeadPosition = new XYZ( 0, 100, 0 );
  options.DimensionLineOrigin = new XYZ( 5, 5, 1 );
  options.DimensionLineDirection = new XYZ( 0, 1, 0 );
  options.DimensionPlaneNormal = view.ViewDirection;
  options.SetElementsToDimension( elementIds );

  using( Transaction tran = new Transaction( doc ) )
  {
    tran.Start( "Create\_Rebar\_Vertical" );

    var mra = MultiReferenceAnnotation.Create(
      doc, view.Id, options );

    var dimension = GetElement<Dimension>(
      doc, mra.DimensionId );

    tran.Commit();
  }
```

Many thanks to Miguel Angel for raising this issue.

#### Case Newsletter

By the way, I really enjoy the
[case newsletter](http://us1.campaign-archive1.com/?u=18e743860774ed83e735552b4&id=e3e9f3aaeb&e=c5bd394c2a) and
the fun items it points to, for instance this
[mental eyeballing exercise](http://woodgears.ca/eyeball/index.html) and
[lawn chair magic video](http://www.youtube.com/embed/p7l3_THa-Yk?rel=0).

Besides the fun stuff, it presents a lot of useful architectural programming news as well.