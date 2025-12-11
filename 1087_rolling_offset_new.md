---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.2
content_type: qa
optimization_date: '2025-12-11T11:44:15.279268'
original_url: https://thebuildingcoder.typepad.com/blog/1087_rolling_offset_new.html
post_number: '1087'
reading_time_minutes: 5
series: general
slug: rolling_offset_new
source_file: 1087_rolling_offset_new.htm
tags:
- csharp
- elements
- filtering
- levels
- parameters
- revit-api
title: Creating a Rolling Offset Pipe Between Two Pipes
word_count: 956
---

### Creating a Rolling Offset Pipe Between Two Pipes

I talked about my nice little MEP related
[CASE BIM Hackathon](http://content.case-inc.com/au-hackathon-2013) project to
[calculate a rolling offset](http://thebuildingcoder.typepad.com/blog/2014/01/calculating-a-rolling-offset-between-two-pipes.html) between
two pipes the day before yesterday.

An obvious immediate question is: OK, and now how do I insert a real pipe segment element instead of just displaying a model line to represent the calculation result?

Actually, this involves two separate steps:

- Create the rolling offset pipe segment
- Connect the three pipes with each other

As I mentioned previously, e.g. discussing
[how to move a duct join](http://thebuildingcoder.typepad.com/blog/2013/10/move-duct-join-add-in-with-video-and-github-support.html),
there are two different approaches to completing these two tasks:

- First place the pipes and then connect their connectors with each other; this automatically generates, places and connects the appropriate fittings.
- Place the fittings, then connect those with each other and the two original pipes; this will automatically generate, place and connect the rolling offset pipe.

Long term, I would like to implement and discuss both of these approaches here.

For the moment, I will just look at the first step, placing a pipe to define the rolling offset, and not worry at all about connecting it yet.

Even just looking at that raises an interesting issue or two.

#### Two Pipe Creation Methods

The Revit API provides two methods to create a new pipe:

- The new static creation method on the class itself: [Pipe.Create](#3).
- The old method on the creation document: [Document.Create.NewPipe.](#4)

#### Initial Problems Using Pipe.Create

I obviously tried to use the new method first.

The static Pipe.Create method to create the pipe takes six arguments:

- document: The document.
- systemTypeId: The id of the piping system type.
- pipeTypeId: The id of the pipe type.
- levelId: The level id for the pipe.
- firstPoint: The first point of the pipe.
- secondPoint: The second point of the pipe.

I tried to grab the required information from one of the two pre-existing pipes and call the creation method like this:

```csharp
  ElementId idSystem = pipe.MEPSystem.Id;
  ElementId idType = pipe.PipeType.Id;
  ElementId idLevel = pipe.LevelId;

  pipe = Pipe.Create( doc, idSystem,
    idType, idLevel, q0, q1 );
```

Unfortunately, this throws an argument exception saying "The systemTypeId is not valid piping system type. Parameter name: systemTypeId".

I tried several other options to retrieve a useable piping system type id, including the ElementId.InvalidElementId and creating a new PipingSystem element, but so far they all failed, unfortunately.

I'll let you know as soon as I have a running solution.

#### Pipe Creation Using NewPipe

As a workaround, I resorted to the old pipe creation method NewPipe provided by the creation document.

I had some old code lying around, so it was no problem reusing that, and it works fine.

The NewPipe method requires the start point, end point, and pipe type.

I use a filtered element collector to retrieve the standard pipe type by comparing its name to "Standard".
I am sure there is also a language independent way of achieving this.

I added an assertion to check that one of the two original pipes is using the same pipe type, so I could also just grab that instead.
That is the case in this super simple sample model.
In the real world, the situation would obviously be more complex.

I also grab the original pipe diameter and set the new one to the same value via its built-in parameter RBS\_PIPE\_DIAMETER\_PARAM.

Here is the resulting code:

```csharp
  BuiltInParameter bip
    = BuiltInParameter.RBS\_PIPE\_DIAMETER\_PARAM;

  double diameter = pipe
    .get\_Parameter( bip ) // "Diameter"
    .AsDouble();

  PipeType pipe\_type\_standard
    = new FilteredElementCollector( doc )
      .OfClass( typeof( PipeType ) )
      .Cast<PipeType>()
      .Where<PipeType>( e
        => e.Name.Equals( "Standard" ) )
      .FirstOrDefault<PipeType>();

  Debug.Assert(
    pipe\_type\_standard.Id.IntegerValue.Equals(
      pipe.PipeType.Id.IntegerValue ),
    "expected all pipes in this simple "
    + "model to use the same pipe type" );

  pipe = doc.Create.NewPipe( q0, q1,
    pipe\_type\_standard );

  pipe.get\_Parameter( bip )
    .Set( diameter );
```

Here is a screen snapshot of the situation with the two pipes before executing the command:

![Two original pipes](img/rolling_offset_pipe_1.png)

The command identifies the two, calculates the rolling offset, adjusts the two existing pipe end points, and generates a new pipe segment between them:

![New rolling offset pipe segment](img/rolling_offset_pipe_2.png)

Note that I have not made any attempt so far to connect the three.

I'll have a look at that some other time.

As promised, I will also try to get the new Pipe.Create method up and running.

The code presented above is available from
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) GitHub
repository, in the external command CmdRollingOffset, and the version discussed here is
[release 2014.0.106.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.106.2).

It implements two Boolean variables to switch between the different tests discussed so far:

- Creation of a model line placeholder
- Pipe creation using Pipe.Create
- Pipe creation using Document.Create.NewPipe

```csharp
  /// <summary>
  /// This command can place either a model line
  /// to represent the rolling offset calculation
  /// result, or insert a real pipe segment and the
  /// associated fittings.
  /// </summary>
  static bool \_place\_model\_line = false;

  /// <summary>
  /// Switch between the new static Pipe.Create
  /// method and the obsolete
  /// Document.Create.NewPipe.
  /// </summary>
  static bool \_use\_static\_pipe\_create = false;
```

I hope you find this useful, and look forward to the further exploration.

If you already know how to achieve some of the missing steps, please feel free to fork and enhance the sample code on GitHub. Thank you!