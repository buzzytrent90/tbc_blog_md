---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.5
content_type: code_example
optimization_date: '2025-12-11T11:44:14.468631'
original_url: https://thebuildingcoder.typepad.com/blog/0725_fam_inst_room_phase.html
post_number: '0725'
reading_time_minutes: 3
series: general
slug: fam_inst_room_phase
source_file: 0725_fam_inst_room_phase.htm
tags:
- elements
- family
- python
- revit-api
- rooms
- schedules
title: Family Instance Room Phase
word_count: 651
---

### Family Instance Room Phase

We already looked at some examples of confusion due to the interaction of phases with the room property on family instances.
Here is a new aspect of this encountered and solved by Patrick Rosendahl of
[SOLAR-COMPUTER GmbH](http://www.solar-computer.de),
who kindly explains the issue and its resolution like this:

**Problem:** I am having a problem with the FamilyInstance Room property.
I looked at the discussion of
[issues with the Room property](http://thebuildingcoder.typepad.com/blog/2011/03/unreliable-room-properties.html) and
[its phase dependency](http://thebuildingcoder.typepad.com/blog/2011/02/phase-dependent-room-properties.html) but
have not been able to solve it yet.

This is the situation:

- Customer makes use of design options and phasing.- Most rooms are in the main model, some rooms are in design options.- There are three phases: phase\_A, phase\_B, phase\_C.- Phase\_C is not used, have not checked the impact if deleted.- Most architecture is created in phase\_A and located in the main model.- Some architecture is created in phase\_B and located in design options.- All family instances and all rooms are created in phase\_B.- Room volume computation is turned on.- Revit's schedules work correctly, i.e. show the correct room-wise list of family instances.

These are the problems encountered:

- Sometimes, the family instance Room property returns null, even though the family instance is clearly within a room.- Even worse: sometimes, the family instance method get\_Room( familyinstance.CreationPhase ) returns null.

Questions:

- Might there be a problem with the design options?- Is there any better workaround than the two approaches mentioned above?- What function is the Revit schedule using to successfully determine the room property of family instances?

As a workaround, I thought of calling the get\_Room method on the family instance in a loop over all available phases in the project, and making use of the document PointInRoom method if that fails.

**Solution:** Here is a promising start for a solution that I am currently exploring:

- Obviously, first try using the family instance Room property directly.- If that fails, try

    ```
    foreach(Phase p) { familyinstance.get_Room(p) }
    ```

    - Hard part: determine primary options;

      ```
      foreach(Room r where r.DesignOption.IsPrimary) {
        r.IsPointInRoom( fi.Location ) }
      ```

      - Emergency:

        ```
        foreach( Room r ) { r.IsPointInRoom( fi.Location ) }
        ```

Here is the actual method implementating this that I am currently working with, including some todo items left as an exercise for the reader ;)
```python
public static Room DetermineRoom( Element el )
{
  FamilyInstance fi = el as FamilyInstance;

  if( fi == null )
  {
    return null;
  }

  // As simple as that?

  try
  {
    if( fi.Room != null )
    {
      //Debug.WriteLine("fi.Room != null");
      return fi.Room;
    }
  }
  catch
  {
  }

  // Try phasing

  Room r = null;

  foreach( Phase p in el.Document.Phases )
  {
    try
    {
      // TODO should check fi.GetPhaseStatus
      // instead of provoking an exception

      r = fi.get\_Room( p );

      if( r != null )
      {
        //Debug.WriteLine("fi.get\_Room( "
        //  + p.Name + ") != null");

        return r;
      }
    }
    catch
    {
    }
  }

  LocationPoint lp = el.Location as LocationPoint;

  if( lp != null )
  {
    // Try design options

    List<Element> roomlst = get\_Elements(
      el.Document, typeof( Room ) );

    // Try rooms from primary design option

    foreach( Element roomel in roomlst )
    {
      Room priroom = roomel as Room;

      if( priroom == null )
        continue;

      if( priroom.DesignOption == null )
        continue;

      if( priroom.DesignOption.IsPrimary )
      {
        // TODO should check whether priroom
        // and el phasing overlaps

        if( priroom.IsPointInRoom( lp.Point ) )
        {
          //Debug.WriteLine(
          //  "priroom.IsPointInRoom != null");

          return priroom;
        }
      }
    }

    // Emergency: try any room

    foreach( Element roomel in roomlst )
    {
      Room room = roomel as Room;

      if( room == null )
        continue;

      // TODO should check whether room
      // and el phasing overlaps

      if( room.IsPointInRoom( lp.Point ) )
      {
        //Debug.WriteLine(
        //  "room.IsPointInRoom != null");
        return room;
      }
    }
  }

  // Nothing found

  return null;
}
```

I think this approach provides is a good starting point.
A more complete solution might possibly take the phase and design option as arguments.
In my context, the function above rarely returns a non-primary design option room.

Many thanks to Patrick for his research and sharing this solution!