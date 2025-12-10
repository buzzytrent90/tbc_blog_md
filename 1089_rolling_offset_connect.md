---
post_number: "1089"
title: "Connecting the Rolling Offset Pipe to its Neighbour Pipes"
slug: "rolling_offset_connect"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'revit-api', 'rooms']
source_file: "1089_rolling_offset_connect.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1089_rolling_offset_connect.html"
---

### Connecting the Rolling Offset Pipe to its Neighbour Pipes

I continue the work on my MEP
[CASE BIM Hackathon](http://content.case-inc.com/au-hackathon-2013) project to
[calculate a rolling offset](http://thebuildingcoder.typepad.com/blog/2014/01/calculating-a-rolling-offset-between-two-pipes.html) between
two pipes and
[insert the rolling offset pipe segment](http://thebuildingcoder.typepad.com/blog/2014/01/calculating-a-rolling-offset-between-two-pipes.html).

The next step consists in connecting the pipes with each other.

Until just now, I was under the false assumption that just connecting the pipes with each other would automatically place the appropriate fittings in between.

That appears to be wrong.

#### MEP Connector Utilities

I created the following new MEP utility methods for The Building Coder samples:

- GetConnectorManager:
  return the given element's connector manager,
  using either the family instance MEPModel property or
  directly from the MEPCurve connector manager
  for ducts and pipes.
- GetConnectorClosestTo:
  return the connector set element
  closest to the given point.
- Connect: connect two MEP elements at a given point.

The API methods required to access a Revit element connector manager differ, depending on whether it is a duct or pipe, i.e., a MEPCurve object, or a fitting, which is a family instance.
The GetConnectorManager helper method wraps and hides this difference to avoid having to remember which access method to use.
This also enables the implementation of generic methods that work on the connectors of all types of MEP elements, such as Connect.

```csharp
  /// <summary>
  /// Return the given element's connector manager,
  /// using either the family instance MEPModel or
  /// directly from the MEPCurve connector manager
  /// for ducts and pipes.
  /// </summary>
  static ConnectorManager GetConnectorManager(
    Element e )
  {
    MEPCurve mc = e as MEPCurve;
    FamilyInstance fi = e as FamilyInstance;

    if( null == mc && null == fi )
    {
      throw new ArgumentException(
        "Element is neither an MEP curve nor a fitting." );
    }

    return null == mc
      ? fi.MEPModel.ConnectorManager
      : mc.ConnectorManager;
  }

  /// <summary>
  /// Return the connector in the set
  /// closest to the given point.
  /// </summary>
  static Connector GetConnectorClosestTo(
    ConnectorSet connectors,
    XYZ p )
  {
    Connector targetConnector = null;
    double minDist = double.MaxValue;

    foreach( Connector c in connectors )
    {
      double d = c.Origin.DistanceTo( p );

      if( d < minDist )
      {
        targetConnector = c;
        minDist = d;
      }
    }
    return targetConnector;
  }

  /// <summary>
  /// Connect the two given elements at point p.
  /// </summary>
  /// <exception cref="ArgumentException">Thrown if
  /// one of the given elements lacks connectors.
  /// </exception>
  public static void Connect(
    XYZ p,
    Element a,
    Element b )
  {
    ConnectorManager cm = GetConnectorManager( a );

    if( null == cm )
    {
      throw new ArgumentException(
        "Element a has no connectors." );
    }

    Connector ca = GetConnectorClosestTo(
      cm.Connectors, p );

    cm = GetConnectorManager( b );

    if( null == cm )
    {
      throw new ArgumentException(
        "Element b has no connectors." );
    }

    Connector cb = GetConnectorClosestTo(
      cm.Connectors, p );

    ca.ConnectTo( cb );
    //cb.ConnectTo( ca );
  }
```

#### Connecting two Pipes Directly

I can make use of the connect method defined above very simply to connect the two original pipes with the rolling offset pipe in between, where the former are provided by the array elements `pipes[0]` and `pipes[1]`, the latter is stored in the variable `pipe`, and q0 and q1 are its start and end points, coinciding with the end and start points of the other two, respectively:

```csharp
  if( null != pipe )
  {
    // Connect rolling offset pipe segment
    // with its neighbours

    Util.Connect( q0, pipes[0], pipe );
    Util.Connect( q1, pipe, pipes[1] );
  }
```

The visual result of running the command is the same as before.
Assume we start off with the two original pipes like this:

![Two original pipes](img/rolling_offset_pipe_1.png)

The rolling offset is placed between them, and their endpoints are moved to match:

![Two original pipe plus rolling offset](img/rolling_offset_pipe_2.png)

Before the Connect method was implemented, I could simply move the pipes apart after creating the rolling offset, and they separate, because they have no relationship with each other:

![Original pipe separated from rolling offset](img/rolling_offset_pipe_3.png)

Implementing and calling the Connect method as shown above to connect the rolling offset with its neighbours ensures that the system remains connected, so moving one of the original pipes away drags the connected rolling offset endpoint along with it:

![Original pipe connected with rolling offset](img/rolling_offset_pipe_4.png)

#### Where are the Fittings?

The resulting system is still far from perfect, though.
Although the pipes are connected with each other, no suitable fittings have been inserted.

I was expecting that to happen automatically.
There are a number of possible reasons why this did not happen, e.g.:

- This is not intended at all.
- The pipes being connected meet at an angle for which no appropriate fitting is available.

The fittings to use for specific pipes are defined by the PipeSettings class.
The description for its GetSpecificFittingAngles method clearly states that "Revit will only use the angles specified during the pipe layout or modifying the pipe layout. When laying out the pipes, if the angle between two pipes is close to the allowed angle, the specific angle is used for that pipe fitting."

So there is still room for further exploration and development.

Anyway, I am perfectly happy to conclude for today with the fact that the pipes have been successfully connected and the helper methods presented above do their job.

The code to connect the rolling offset is included in the
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) GitHub
repository, in the external command CmdRollingOffset, and the version discussed here is
[release 2014.0.106.3](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2014.0.106.3).