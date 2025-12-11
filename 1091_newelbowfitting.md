---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.4
content_type: qa
optimization_date: '2025-12-11T11:44:15.286651'
original_url: https://thebuildingcoder.typepad.com/blog/1091_newelbowfitting.html
post_number: '1091'
reading_time_minutes: 4
series: general
slug: newelbowfitting
source_file: 1091_newelbowfitting.htm
tags:
- csharp
- family
- parameters
- references
- revit-api
- selection
title: Simpler Rolling Offset Using NewElbowFitting
word_count: 762
---

### Simpler Rolling Offset Using NewElbowFitting

I continue the work on my MEP
[CASE BIM Hackathon](http://content.case-inc.com/au-hackathon-2013) project to
[calculate a rolling offset](http://thebuildingcoder.typepad.com/blog/2014/01/calculating-a-rolling-offset-between-two-pipes.html) between
two pipes,
[insert the rolling offset pipe segment](http://thebuildingcoder.typepad.com/blog/2014/01/calculating-a-rolling-offset-between-two-pipes.html),
[connect it to its neighbours](http://thebuildingcoder.typepad.com/blog/2014/01/connecting-the-rolling-offset-pipe-to-its-neighbour-pipes.html) and
[explicitly placing elbow fittings](http://thebuildingcoder.typepad.com/blog/2014/01/connecting-the-rolling-offset-pipe-to-its-neighbour-pipes.html).

The explicit fitting placement was pretty hard and fiddly to implement, requiring the following steps:

- Explore a manually generated model to determine the appropriate family and elbow fitting types.
- Research the fitting family instance parameters to determine how to define its diameter and elbow angle.
- Explicitly place elbow fitting family instance using the NewFamilyInstance method.
- Orient it in the proper direction toward the other, parallel, offset pipe.
- Shorten the original pipes to make space for the fitting.
- Create the connections between neighbouring pipes and fittings.

It worked, but only after a lot of research and effort.

As pointed out by Tate in the
[comment](http://thebuildingcoder.typepad.com/blog/2014/01/connecting-the-rolling-offset-pipe-to-its-neighbour-pipes.html?cid=6a00e553e168978833019b04c5511d970d#comment-6a00e553e168978833019b04c5511d970d) on
[connecting the rolling offset pipe](http://thebuildingcoder.typepad.com/blog/2014/01/connecting-the-rolling-offset-pipe-to-its-neighbour-pipes.html) to
its neighbours, almost all that effort can be avoided by using the NewElbowFitting method instead.
It selects, creates, adapts, places and connects an elbow fitting automatically, with the additional advantage of taking all the Revit pipe routing preferences and other settings into account.

We touched on the topic of new fitting creation methods now and then in the past, but never presented an example using NewElbowFitting in context like this, so it is actually rather overdue:

- [Cable tray orientation and fittings](http://thebuildingcoder.typepad.com/blog/2010/05/cable-tray-orientation-and-fittings.html)
- [Use of NewTakeOffFitting on a duct](http://thebuildingcoder.typepad.com/blog/2011/02/use-of-newtakeofffitting-on-a-duct.html)
- [Set elbow fitting type](http://thebuildingcoder.typepad.com/blog/2011/02/set-elbow-fitting-type.html)
- [Use of NewTakeOffFitting on a pipe](http://thebuildingcoder.typepad.com/blog/2011/04/use-of-newtakeofffitting-on-a-pipe.html)
- [MEP placeholders](http://thebuildingcoder.typepad.com/blog/2011/07/mep-placeholders.html)
- [Elbow fitting selection and dimensioning](http://thebuildingcoder.typepad.com/blog/2012/07/elbow-fitting-selection-and-dimensioning.html)

#### Rolling Offset Fitting Placement Using NewElbowFitting

I expanded my previous code that calculates and places the rolling offset pipe segment.

Last time I discussed it, it
[connected a pipe directly](http://thebuildingcoder.typepad.com/blog/2014/01/connecting-the-rolling-offset-pipe-to-its-neighbour-pipes.html) to
the neighbouring original pipes with no fittings in between.

To make use of NewElbowFitting, I simply replaced the code hooking up the connectors by six new lines that (twice) select the appropriate connector pairs and invoke the method like this:

```csharp
  pipe = doc.Create.NewPipe( q0, q1,
    pipe\_type\_standard );

  pipe.get\_Parameter( bipDiameter )
    .Set( diameter );

  // Connect rolling offset pipe segment
  // directly with the neighbouring original
  // pipes
  //
  //Util.Connect( q0, pipes[0], pipe );
  //Util.Connect( q1, pipe, pipes[1] );

  // NewElbowFitting performs the following:
  // - select appropriate fitting family and type
  // - place and orient a family instance
  // - set its parameters appropriately
  // - connect it with its neighbours

  Connector con0 = Util.GetConnectorClosestTo(
    pipes[0], q0 );

  Connector con = Util.GetConnectorClosestTo(
    pipe, q0 );

  doc.Create.NewElbowFitting( con0, con );

  Connector con1 = Util.GetConnectorClosestTo(
    pipes[1], q1 );

  con = Util.GetConnectorClosestTo(
    pipe, q1 );

  doc.Create.NewElbowFitting( con, con1 );
```

As you can see, all the questionable hard-coded calculations required to explicitly place the fittings are now eliminated.

The resulting model looks exactly the same as the one presented yesterday, generated by
[explicitly placing the elbow fittings](http://thebuildingcoder.typepad.com/blog/2014/01/explicitly-placing-rolling-offset-pipe-elbow-fittings.html#4).

The new method to place the elbow fittings and properly connect the rolling offset is included in
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) GitHub
repository, in the external command CmdRollingOffset, and the version discussed here is
[release 2014.0.106.5](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.106.5).

That just about concludes this project, except for the question on how to use Pipe.Create instead of Document.Create.NewPipe, which is still open.