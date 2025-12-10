---
post_number: "0107"
title: "Create Room on Level in Phase"
slug: "room_on_level_in_phase"
author: "Jeremy Tammik"
tags: ['elements', 'geometry', 'levels', 'parameters', 'revit-api', 'rooms', 'schedules', 'transactions', 'views']
source_file: "0107_room_on_level_in_phase.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0107_room_on_level_in_phase.html"
---

### Create Room on Level in Phase

**Question:**
I want to create a room at a specific location on a given level and in a specific phase as well.
I see a constructor to create a room in a phase, and a constructor to create a room in a level at given UV coordinates, but there is no constructor to do both.
If I create a room on a level, I don't see how to establish the phase.
If I create it in a phase, I don't see how to position it on a level at the required position.
Is there a way to achieve both?

**Answer:**
First, here is some background information on the relevant room properties and existing samples that create rooms:

#### Room Level, Location and Phase Properties

There are is an SDK sample RoomSchedule and a Revit API introduction training lab Lab2\_0\_CreateLittleHouse which show how to place a room, but neither of them demonstrate setting all three of its level, location and phase properties.

The following properties allow you to get its level and get and set its location:

- Level: returns the level to which the room is assigned.
- Location: returns a location point object. The point within the location object can be get and set. This is the location of the room within the level. The Z location should be the elevation of the level and is not changeable.

You can also read its level by using the LEVEL\_NAME or ROOM\_LEVEL\_ID parameters. Unfortunately, these parameters are both read-only:

```
LEVEL_NAME     String/Text  read-only  "Level 1"
ROOM_LEVEL_ID  ElementId    read-only  Levels 13071 'Level 1'
```

It has properties to read its creation and demolishment phases, and they are also marked as read-only:

- PhaseCreated: the phase when the element is created.
- PhaseDemolished: the phase when the element is to be demolished.

So I do not see any way to directly modify the level or phase properties of an existing room.

#### Ideas for creating a new Room with specific Level and Phase

The following overloads can be used to create a new room or place an existing one:

```
  Room NewRoom( Phase );
  Room NewRoom( Level, UV point );
  Room NewRoom( Room, PlanCircuit );
```

A previous post on
[plan topology](http://thebuildingcoder.typepad.com/blog/2009/01/plantopology-class.html)
and the
[CmdPlanTopology command](http://thebuildingcoder.typepad.com/blog/2009/01/plantopology-class.html)
shows how to use the latter overload to create a new room.

You can also use it to place an existing unplaced room. You should be able to use the first overload to create a new room in a specified phase and then feed the newly created room to the third overload in order to place it.

Here is suggestion for another even simpler approach:

The NewRoom( Level, UV ) method uses the current phase. You could get the current phase of your current view, store that, change the current phase to whatever you like, call NewRoom, and then set the phase back to its original value again. I havent tried it, but it seems like that should work.

#### Difficulties creating a new Room with specific Level and Phase

Here are the results of trying both approaches:

**1.**
I tried to change the Phase of the actual view before creating the new Room but that does not work.
I changed the Phase like this:

```
View v = document.ActiveView;
ElementId id = myPhase.Id;
Parameter p = v.get_Parameter(
  BuiltInParameter.VIEW_PHASE );
p.Set( ref id );
Room r = doc.NewRoom( level, uv );
```

After executing the command, I can see the phase of the view has changed, but the room is not created in the phase I set.
I tried to put the code changing the phase in a transaction, too, and that doesn't work either.

**2.**
Then I created the room using the method NewRoom( Phase ).
And indeed, I can position the resulting room with the second method, NewRoom( Room, PlanCircuit ).
But this brings up a new problem: how can I determine which circuit to use to create my room at the given UV coordinates?
I could not find an easy way to do this.
Finally I ended up iterating through all the circuits in my level and phase.
If the circuit has no room, I create the room, and test if the UV point (with the level's Z coordinate) lies within its bounding box.
If so, we are done.
If not, delete the room, and continue trying with the next circuit.
Rather inefficient:

```
// create the room in the phase:

Room roomNueva = doc.Create.NewRoom( phase );

// search for the plan circuit which
// locates the room at the given UV coordinates:

XYZ xyz = new XYZ( uv.U, uv.V, level.Elevation );
PlanTopology pt = doc.get_PlanTopology( level, phase );
foreach( PlanCircuit pc in pt.Circuits )
{
  if( !pc.IsRoomLocated )
  {
    Room room = doc.Create.NewRoom( roomNueva, pc );

    // test if the bounding box contains our point:

    if( Geometry.BoundingBoxXYZContains(
      roomNueva.get_BoundingBox( null ),
      xyz ) )
    {
      break;
    }
    else // if not, delete the room:
    {
      doc.Delete( room );
      room = null;
    }
  }
}
```

After some discussion with the Revit development team, the conclusion is that probably the 'right' way to go about this would be to create the room and then set the phase, which the API currently does not let you do.
The approach iterating through all the plan circuits and testing where the room ends up for each seems to be the only way to go for now.