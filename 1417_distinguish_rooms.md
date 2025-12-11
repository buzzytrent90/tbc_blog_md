---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.8
content_type: qa
optimization_date: '2025-12-11T11:44:15.973453'
original_url: https://thebuildingcoder.typepad.com/blog/1417_distinguish_rooms.html
post_number: '1417'
reading_time_minutes: 4
series: general
slug: distinguish_rooms
source_file: 1417_distinguish_rooms.md
tags:
- csharp
- elements
- filtering
- revit-api
- rooms
- schedules
- sheets
title: Distinguish Rooms
word_count: 753
---

### How to Distinguish Redundant Rooms
A couple of interesting Revit API issues were resolved during my recent absence.
Let's start with this question raised by Miroslav Schonauer and resolved by Diane Christoforo:
\*\*Question:\*\* Using the terminology as shown in Schedules, I need to report all 'Not Placed', 'Redundant' and 'Not Enclosed' rooms.
I cannot rely on any particular schedule being present in the model.
So far, my research shows that:
- All of these 3 'failure' cases have `.Area` as 0.
- 'Not Placed' additionally has `.Location` as null.
The only remaining issue is how to distinguish between 'Redundant' and 'Not Enclosed' rooms.
Do you have any ideas how can I do this comprehensively and deterministically from the room properties and params?
BTW, I'm aware that an alternative approach would be via the Failure API. I can see, e.g., the enumeration value `BuiltInFailures.RoomFailures.RoomNotEnclosed`.
After just reading about it and looking into some samples, I worry about this approach and think that it is possible to 'override' the failures, which may prevent me from determining whether the rooms really are Not Enclosed or Redundant.
Here is sample code illustrating the approach I have been able to develop so far:

```
  ///
  /// Distinguish 'Not Placed', 'Redundant' and 'Not Enclosed' rooms.
  ///
  void DistinguishRooms(
    Document doc,
    ref StringBuilder sb,
    ref int numErr,
    ref int numWarn )
  {
    sb = new StringBuilder();

    FilteredElementCollector rooms
      = new FilteredElementCollector( doc );

    rooms.WherePasses( new RoomFilter() );

    foreach( Room r in rooms )
    {
      sb.AppendFormat( "\r\n  Room {0}:'{1}': ",
        r.Id.ToString(), r.Name );

      if( r.Area > 0 ) // OK if having Area
      {
        sb.AppendFormat( "OK (A={0}[ft3])", r.Area );
      }
      else if( null == r.Location ) // Unplaced if no Location
      {
        sb.AppendFormat( "UnPlaced (Location is null)" );
      }
      else
      {
        sb.AppendFormat( "NotEnclosed or Redundant "
          + "- how to distinguish?" );
      }
    }
  }
```

Any ideas if this is possible?
\*\*Answer:\*\* Rooms that are redundant still have normal boundaries.
So you can call `Room.GetBoundarySegments` to test.
The list will be empty for a non-enclosed or unplaced room.
\*\*Response:\*\* Thank you, Diane!
Here is the final solution implementing your suggestion:

```
  public enum RoomState
  {
    Unknown,
    Placed,
    NotPlaced,
    NotEnclosed,
    Redundant
  }

  ///
  /// Distinguish 'Not Placed',  'Redundant'
  /// and 'Not Enclosed' rooms.
  ///
  RoomState DistinguishRoom( Room room )
  {
    RoomState res = RoomState.Unknown;

    if( room.Area > 0 )
    {
      // Placed if having Area

      res = RoomState.Placed;
    }
    else if( null == room.Location )
    {
      // No Area and No Location => Unplaced

      res = RoomState.NotPlaced;
    }
    else
    {
      // must be Redundant or NotEnclosed

      SpatialElementBoundaryOptions opt
        = new SpatialElementBoundaryOptions();

      IList<IList<BoundarySegment>> segs
        = room.GetBoundarySegments( opt );

      res = ( null == segs || segs.Count == 0 )
        ? RoomState.NotEnclosed
        : RoomState.Redundant;
    }
    return res;
  }
```

Jeremy adds: I added Miro's test methods as `DistinguishRoomsDraft` and `DistinguishRoom`
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2016.0.126.8](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2016.0.126.8) in
the module [CmdListAllRooms.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdListAllRooms.cs#L30-L110).
Many thanks to Miro and Diane for the interesting question, research and answer.
I actually cleaned up both the draft and final methods before publishing them, with the two following strong recommendations
on [performance](#2) and [exception handling](#3):
#### Performance
Looking at your code, I notice an important inefficiency:

```
  foreach( Element elRoom in fecRooms.ToElements() )
  {
    Room r = elRoom as Room;
```

You can save yourself the duplication of the entire list of rooms by eliminating the call to `ToElements`, and the cast is not needed either:

```
  foreach( Room r in fecRooms )
```

I hope this helps... systematically!
I hope you can eliminate a thousand calls to `ToElements` all over all your projects.
I discussed this and other similar issues many times here in the past, e.g.
for [FindElement and filtered element collector optimisation](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html)
and
[ToElementIds performance](http://thebuildingcoder.typepad.com/blog/2012/12/toelementids-performance.html).
#### Exception Handling
I eliminated a catch-all exception handler from Miro's solution.
Are you aware of the strong recommendation against never, ever, catching all exceptions?
Using a `catch` with no specific exceptions listed is very strongly discouraged, cf. e.g. these articles
on [C# catching all exceptions](http://stackoverflow.com/questions/315948/c-catching-all-exceptions)
and [why `catch(Exception)` and empty `catch` is bad](http://blogs.msdn.com/b/dotnet/archive/2009/02/19/why-catch-exception-empty-catch-is-bad.aspx).
Here is another more extensive discussion
on [exception handling in C# with the "Do Not Catch Exceptions That You Cannot Handle" rule in mind](http://www.codeproject.com/Articles/7557/Exception-Handling-in-C-with-the-quot-Do-Not-Catch).