---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.3
content_type: qa
optimization_date: '2025-12-11T11:44:15.811896'
original_url: https://thebuildingcoder.typepad.com/blog/1343_vacation.html
post_number: '1343'
reading_time_minutes: 7
series: general
slug: vacation
source_file: 1343_vacation.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- geometry
- parameters
- references
- revit-api
- transactions
- views
- walls
- windows
title: Clicks, DMU, Surfaces, FireRating Feedback, Vacation
word_count: 1427
---

### Clicks, DMU, Surfaces, FireRating Feedback, Vacation

Let me list some of the topics we discussed today in the Revit API discussion forum before I head off for vacation:

- [Reacting to Windows mouse clicks in Revit](#2).
- [Reacting to changes and setting parameters using DMU or DocumentChanged](#3).
- [Creating a surface in Revit](#4).
- [FireRating in the Cloud feedback](#5).
- [Vacation Time Now](#6).

Summer has seriously arrived in Europe, and the weather is really good as well, for a change.

I took advantage of it this weekend and climbed the Alphubel (4201 m) from the Täschhütte:

![Alphubel 4201 m](img/alphubeljoch_alphubel.jpg)

We planned to go up the standard route over the SE ridge and then took the E face instead.
Unfortunately, we only stayed on the summit for about 20 minutes, admiring the splendid view, hurrying down again fast to avoid the snow getting too soft in the sunshine for a comfortable descent.

Before I leave, let's highlight a couple of the topics we discussed today on the
[Revit API discussion forum](http://forums.autodesk.com/t5/revit-api/bd-p/160).

#### Reacting to Windows Mouse Clicks in Revit

**Question:**
I am thinking about some Revit API such that I can select a lamp, and using double click or right click with the mouse to do something.

Is there some function can be used for the application above?

**Answer:**
The Revit API does not provide any specific support for this.

Also, I am not sure whether it will help you users if you modify the behaviour of the standard Revit user interface.
I would suggest always working with the system, not against it.
Note that
[Revit has a very clear idea of best practice and optimal workflows](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.41).

Still, there is nothing to stop you from using the Windows and .NET API to determine the single and double clicks and doing whatever you like with them.

However, always be aware that you need to be inside a valid Revit API context to make calls to the Revit API.

If you use the Windows API to determine a mouse click, you will not be in a valid Revit API context; you will basically be in a
[modeless context and driving Revit from outside](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28) if you wish to access the API.

It is possible to determine what element is under the mouse cursor, once you have access to Revit API functionality, e.g., using the
[UIView, Windows coordinates and ReferenceIntersector](http://thebuildingcoder.typepad.com/blog/2012/10/uiview-windows-coordinates-referenceintersector-and-my-own-tooltip.html).

Good luck playing with this!

#### Reacting to Changes and Setting Parameters using DMU or DocumentChanged

**Question:**
Last week I was trying to develop some extra functionality within Revit that automatically registers when and who adds or modifies family types and instances. To realize this I introduced some project parameters that are assigned to the specific families. What I am trying to do is that when something is added or modified, the parameters automatically are set. So far, so good.

I've tried multiple solutions but until now I just can't figure out the last step. To retrieve the element id's I've tried a DocumentChanged event that does find the right id's but, because the event isn't meant for changing or manipulating elements in any way whatsoever, I got stuck.

Another solution I've tried is the use of IUpdaters. When it comes to modifying it does the job fairly good job. However, copying elements doesn't work seem to work properly: when I copy an instance, the parameters of the newly created element are set to the values of the original element. Besides this issue I stumbled upon a bigger problem: adding instances. Where the DocumentChanged event retrieves all the elements, the IUpdater can't retrieve the last element because the timing comes in a bit earlier than the DocumentChanged event. Example: when I'm placing a wall chain of three walls, the first two walls are set properly but the parameters of the last one comes empty.

My question: how can I retrieve all added and modified elements (or corresponding id's) and change there associated parameters?

**Answer:**
I think you are absolutely on the right track.

You are apparently missing one really fundamental important piece, though:

The
[dynamic model updater or DMU](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.31) interaction
happens in the same transaction as the operation that triggers it.

The document changed event is called after the transaction has been closed.

Also, the latter can no longer make any changes to the model; it is a read-only event.

If you need to collect information from both of these events – which you apparently do – you certainly can, no problem whatsoever.

One possibility to afterwards process and save this info in parameters on the modified and copied elements would be to temporarily subscribe to the
[Idling event](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.28).

In the Idling event handler, you can modify the model again as you wish, i.e., save your data, and also immediately unsubscribe from the Idling event again.

Here is a dedicated discussion on
[DocumentChanged versus dynamic model updater](http://thebuildingcoder.typepad.com/blog/2012/06/documentchanged-versus-dynamic-model-updater.html).

#### Creating a Surface in Revit

**Question:**
I need to create a surface in Revit API with some points. The surface can be anything, like a roof, a floor, or another component. The surface is not plane. Is there any command that can do this? Or I need to find a way to divide the surface in little plans?

**Answer:**
Here is a solution for create a floor from points:

```csharp
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Document doc = uidoc.Document;

  using( Transaction tx = new Transaction( doc ) )
  {
    tx.Start( "Create a Floor" );

    int n = 4;
    XYZ[] points = new XYZ[n];
    points[0] = XYZ.Zero;
    points[1] = new XYZ( 10.0, 0.0, 0.0 );
    points[2] = new XYZ( 10.0, 10.0, 0.0 );
    points[3] = new XYZ( 0.0, 10.0, 0.0 );

    CurveArray curve = new CurveArray();

    for( int i = 0; i < n; i++ )
    {
      Line line = Line.CreateBound( points[i],
        points[(i < n-1) ? i + 1 : 0] );

      curve.Append( line );
    }

    doc.Create.NewFloor( curve, true );

    tx.Commit();
  }
  return Result.Succeeded;
```

Note that the document modification happens within a transaction, and the transaction is encapsulated in and handled by a `using` statement, because
[using `using` automagically disposes and rolls back](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html).

The Building Coder provides a similar method to
[create a FilledRegion](http://thebuildingcoder.typepad.com/blog/2013/07/create-a-filled-region-to-use-as-a-mask.html).

You have a quite large number of options, really.

It depends on what you really need, of course.

You can also start by looking at the
[form creation API](http://thebuildingcoder.typepad.com/blog/2009/07/revit-form-creation-api.html).

That is pretty old, though.

The newest and maybe easiest option might be
[using the DirectShape element](http://thebuildingcoder.typepad.com/blog/2015/02/from-hack-to-app-obj-mesh-import-to-directshape.htm).

Nowadays,
[DirectShape is one of the easiest methods to create and visualise geometry](http://thebuildingcoder.typepad.com/blog/2015/07/intersect-solid-filter-avf-and-directshape-for-debugging.html#5) in a Revit model.

So maybe I would actually start by looking at that.

#### FireRating in the Cloud Feedback

I received some really nice feedback from
[@TroyGates](https://twitter.com/troygates) on the
[FireRating in the Cloud](https://github.com/jeremytammik/FireRatingCloud) project that I would like to share:

> Check out
> [@jeremytammik](https://twitter.com/jeremytammik)
> [#Revit](https://twitter.com/hashtag/revit) API
> example for automating door fire rating parameters from a database
> [FireRating in the Cloud](http://thebuildingcoder.typepad.com/blog/2015/07/grevit-firerating-in-the-cloud-demo-deployment-vacation.html#2)
>
> @jeremytammik your fire rating db gives me so many ideas for future api projects, thanks!!

That was indeed my intention!

Many thanks, Troy, for your nice appreciation!

I hope this helps inspire many others as well.

#### Vacation Time Now

On that happy note, let me take my leave...

I'll be away on vacation for the next few weeks, relaxing in Switzerland and climbing another mountain or two, hopefully.

I wish you a wonderful summer!