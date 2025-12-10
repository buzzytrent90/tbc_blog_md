---
post_number: "1233"
title: "NewCrossFitting Connection Order"
slug: "newcrossfitting"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'parameters', 'python', 'revit-api', 'selection', 'transactions']
source_file: "1233_newcrossfitting.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/1233_newcrossfitting.html"
---

### NewCrossFitting Connection Order

A long, long time ago, Davex raised a very pertinent question on the error message and
[nondescriptive exception thrown](http://forums.autodesk.com/t5/revit-api/newcrossfitting-nondescriptive-exception-thrown/td-p/2824298) by
the attempted creation of a new Revit MEP 2014 cross fitting.

I finally get around to addressing that.
I very much hope that Davex also figured it out in the meantime...

**Question:** I am attempting to create a cross fitting by calling Owner.Document.Create.NewCrossFitting, passing in 4 connectors to pipes that are on the same plane and lie along orthogonal lines.
I've verified that the connectors are all at the same point.

When I try to call NewCrossFitting, an InvalidOperationException is thrown, with a message stating the obvious:

```
  "failed to insert cross"
```

Are there many scenarios where this exception can be thrown, or does it have one specific meaning? If the former, is there a way to make a feature request of the Revit developers to write exception descriptions that aren't totally useless?

By the way, many other Revit MEP API method descriptions in the Revit API help file RevitAPI.chm and the developer guide are also very short indeed.
I hope that more information about them will be made available one of these days.

**Answer:** The API expects the four cross fitting connectors to be specified in the order main – main – side - side. When the input pipes are selected using the PickElementsByRectangle method, they end up cross selected, and the connection order passed in is main-side-main-side. Once you switch the order back to the expected sequence, the cross fitting will be successfully added.

I agree that the exception messages could be improved to warn users about these kinds of expectations. Part of problem is that in the distant past, these handwritten APIs were often created separately from the internal source code. It is a long overdue project to remove all the handwritten APIs.

**Response:** Thanks very much for pointing to the way out.
This works like a charm.

I once tried to swap the second with the fourth parameters and that failed as well, so I never tried the other order.

The secret is indeed the parameter order: main-main-side-side.

Thanks to Joe Ye and Liang Zhu for clarifying this.

#### The Building Coder Sample CmdNewCrossFitting Command

I added the solution to The Building Coder samples as a new external command CmdNewCrossFitting.

Among other things, it demonstrates use of a newly implemented Element extension method GetCurve, the PickElementsByRectangle selection functionality, ensuring that exactly 2, 3 or 4 pipes are selected, and last but not least, the NewTeeFitting and NewCrossFitting methods:

- [The Element extension method GetCurve](#3)
- [Pipe direction](#4)
- [Parallel pipes predicate](#5)
- [CmdNewCrossFitting implementation](#6)
- [Results](#7)
- [Download](#8)

By the way, the NewElbowFitting that is also used below was already exercised by the CmdRollingOffset command implemented to
[explicitly place rolling offset elbow fittings](http://thebuildingcoder.typepad.com/blog/2014/01/explicitly-placing-rolling-offset-pipe-elbow-fittings.html).

#### The Element Extension Method GetCurve

I implemented a new extension method for the Revit API Element class to provide direct access to the geometrical Curve element encapsulated in the Element Location property, if it exists:

```python
public static class JtElementExtensionMethods
{
  /// <summary>
  /// Return the curve from a Revit database Element
  /// location curve, if it has one.
  /// </summary>
  public static Curve GetCurve( this Element e )
  {
    Debug.Assert( null != e.Location,
      "expected an element with a valid Location" );

    LocationCurve lc = e.Location as LocationCurve;

    Debug.Assert( null != lc,
      "expected an element with a valid LocationCurve" );

    return lc.Curve;
  }
}
```

#### Pipe Direction

The analysis of the geometrical pipe relationships with each other obviously requires us to determine their direction. That is provided by the following trivial method:

```csharp
  /// <summary>
  /// Return the normalised direction of the given pipe.
  /// </summary>
  XYZ GetPipeDirection( Pipe pipe )
  {
    Curve c = pipe.GetCurve();
    XYZ dir = c.GetEndPoint( 1 ) - c.GetEndPoint( 1 );
    dir = dir.Normalize();
    return dir;
  }
```

#### Parallel Pipes Predicate

A similarly trivial comparison of the directions of two given pipes tells us whether they are parallel or not:

```csharp
  /// <summary>
  /// Are the two given pipes parallel?
  /// </summary>
  bool IsPipeParallel( Pipe p1, Pipe p2 )
  {
    Line c1 = p1.GetCurve() as Line;
    Line c2 = p2.GetCurve() as Line;
    return Math.Sin( c1.Direction.AngleTo(
      c2.Direction ) ) < 0.01;
  }
```

#### CmdNewCrossFitting Implementation

With these tools in hand, we are now ready for the interesting part.

We prompt the user to select 2, 3 or 4 pipes.

The selection process continues until she succeeds in doing so or cancels, in which case the ensuing exception is handled gracefully.

The number of pipes selected determines whether an elbow, tee or cross fitting is inserted.

The geometric relationships between the pipes are used to determine the correct order to pass in the connectors to the respective creation methods.

```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication app = commandData.Application;
  UIDocument uidoc = app.ActiveUIDocument;
  Document doc = uidoc.Document;

  IList<Element> pipes = null;
  int n = 0;

  // Ensure that 2, 3 or 4 pipes are selected.

  while( n < 2 || 4 < n )
  {
    if( 0 != n )
    {
      Util.InfoMsg( string.Format(
        "You picked {0} pipe{1}. "
        + "Please only pick 2, 3 or 4.",
        n, Util.PluralSuffix( n ) ) );
    }

    try
    {
      Selection sel = app.ActiveUIDocument.Selection;

      ISelectionFilter filter = new CmdRollingOffset
        .PipeElementSelectionFilter();

      pipes = sel.PickElementsByRectangle(
        filter, "Please pick some pipes." );
    }
    catch( Autodesk.Revit.Exceptions
      .InvalidOperationException )
    {
      return Result.Cancelled;
    }
    n = pipes.Count;
  }

  XYZ pt = null;

  using( Transaction tx = new Transaction( doc ) )
  {
    tx.Start( "CreateConnector" );
    if( pipes.Count() <= 1 )
      return Result.Cancelled;

    Pipe pipe1 = pipes[0] as Pipe;
    Pipe pipe2 = pipes[1] as Pipe;

    Curve curve1 = pipe1.GetCurve();
    Curve curve2 = pipe2.GetCurve();

    XYZ p1 = curve1.GetEndPoint( 0 );
    XYZ q1 = curve1.GetEndPoint( 1 );

    XYZ p2 = curve2.GetEndPoint( 0 );
    XYZ q2 = curve2.GetEndPoint( 1 );

    if( q1.DistanceTo( p2 ) < 0.1 )
    {
      pt = ( q1 + p2 ) \* 0.5;
    }
    else if( q1.DistanceTo( q2 ) < 0.1 )
    {
      pt = ( q1 + q2 ) \* 0.5;
    }
    else if( p1.DistanceTo( p2 ) < 0.1 )
    {
      pt = ( p1 + p2 ) \* 0.5;
    }
    else if( p1.DistanceTo( q2 ) < 0.1 )
    {
      pt = ( p1 + q2 ) \* 0.5;
    }
    else
    {
      message = "Please select two pipes "
        + "with near-by endpoints.";

      return Result.Failed;
    }

    Connector c1 = Util.GetConnectorClosestTo(
      pipe1, pt );

    Connector c2 = Util.GetConnectorClosestTo(
      pipe2, pt );

    if( pipes.Count() == 2 )
    {
      if( IsPipeParallel( pipe1, pipe2 ) == true )
      {
        doc.Create.NewUnionFitting( c1, c2 );
      }
      else
      {
        doc.Create.NewElbowFitting( c1, c2 );
      }
    }
    else if( pipes.Count() == 3 )
    {
      Pipe pipe3 = pipes[2] as Pipe;

      XYZ v1 = GetPipeDirection( pipe1 );
      XYZ v2 = GetPipeDirection( pipe2 );
      XYZ v3 = GetPipeDirection( pipe3 );

      Connector c3 = Util.GetConnectorClosestTo(
        pipe3, pt );

      if( Math.Sin( v1.AngleTo( v2 ) ) < 0.01 ) //平行
      {
        doc.Create.NewTeeFitting( c1, c2, c3 );
      }
      else //v1, 和v2 垂直.
      {
        if( Math.Sin( v3.AngleTo( v1 ) ) < 0.01 ) //v3, V1 平行
        {
          doc.Create.NewTeeFitting( c3, c1, c2 );
        }
        else //v3, v2 平行
        {
          doc.Create.NewTeeFitting( c3, c2, c1 );
        }
      }
    }
    else if( pipes.Count() == 4 )
    {
      Pipe pipe3 = pipes[2] as Pipe;
      Pipe pipe4 = pipes[3] as Pipe;

      Connector c3 = Util.GetConnectorClosestTo(
        pipe3, pt );

      Connector c4 = Util.GetConnectorClosestTo(
        pipe4, pt );

      //以从哪c1为入口.

      // The required connection order for a cross
      // fitting is main – main – side - side.

      if( IsPipeParallel( pipe1, pipe2 ) )
      {
        doc.Create.NewCrossFitting(
          c1, c2, c3, c4 );
      }
      else if( IsPipeParallel( pipe1, pipe3 ) )
      {
        try
        {
          doc.Create.NewCrossFitting(
            c1, c3, c2, c4 );
        }
        catch( Exception ex )
        {
          TaskDialog.Show(
            "Cannot insert cross fitting",
            ex.Message );
        }
      }
      else if( IsPipeParallel( pipe1, pipe4 ) )
      {
        doc.Create.NewCrossFitting(
          c1, c4, c2, c3 );
      }
    }
    tx.Commit();
  }
  return Result.Succeeded;
}
```

#### Result

Here is the initial starting point with four perpendicular pipes meeting in a common point:

![Four pipes meeting in a point](img/cross_fitting_pipes.png)

The result of running the CmdNewCrossFitting command and selecting all four pipes now looks like this:

![The resulting cross fitting](img/cross_fitting_result.png)

#### Download

The most up to date version of The Building Coder samples is always provided in
[its GitHub repository](https://github.com/jeremytammik/the_building_coder_samples),
and the version described above is
[release 2015.0.115.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2015.0.115.0).