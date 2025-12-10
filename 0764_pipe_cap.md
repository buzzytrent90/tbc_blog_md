---
post_number: "0764"
title: "Create a Pipe Cap"
slug: "pipe_cap"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'levels', 'parameters', 'references', 'revit-api', 'views']
source_file: "0764_pipe_cap.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0764_pipe_cap.html"
---

### Create a Pipe Cap

Happily mixing the different Revit versions, let me follow up my work on the
[Revit MEP 2013 API](http://thebuildingcoder.typepad.com/blog/2012/05/the-revit-2013-mep-api-and-external-services.html) and the
[updated AdnRme sample](http://thebuildingcoder.typepad.com/blog/2012/05/the-adn-mep-sample-adnrme-for-revit-mep-2013.html) by
another MEP related topic.

Before getting to that, though, here is a brief heads-up for a new technology preview on Autodesk Labs:

#### BIM Coordinator

The effective organization of project data in shared or related coordinates is essential to effective collaboration across disciplines and good quality project information.
[BIM Coordinator](http://labs.autodesk.com/utilities/bim_coordinator) for
AutoCAD Civil 3D and Revit is a technology preview that assists project team members with building and site grids.

Here is a brief overview taken from the labs:

**Building grids** are generally selected based on the layout and shape of the building, usually using the lower left corner of the selected orientation as a reference point. Building grids are established by the lead designer of a project for use by all members of the building design team. This grid ensures that different disciplines' building designs use the same reference for purposes of building analyses, drawing production, and other tasks requiring coordinated building data.

**Site grids** are generally based on Northing and Easting coordinates. They are usually established for a region; although arbitrary grids can be established for a single project or site. For example, in the UK most sites are related to the Ordinance Survey (OS) grid. Site elevations (levels) are also based on an OS benchmark or other site/project benchmark.

[BIM Coordinator](http://labs.autodesk.com/utilities/bim_coordinator) automates the more error-prone manual steps in the coordination of site and building grids between the AutoCAD and Revit environments, currently for the 2012 versions. The tool specifies one common XYZ point and its orientation in the XY plane between the two coordinate systems. This reference point defines the relationship between the building and site grid.

Miroslav Schonauer will be explaining some background and sharing source code from this utility at the upcoming
[AEC DevCamp](http://thebuildingcoder.typepad.com/blog/2012/02/aec-devcamp-coming-up-again.html) in
Waltham June 6-7.

#### Create a Pipe Cap

Back to the Revit MEP API, I presented an example to
[create a pipe cap](http://thebuildingcoder.typepad.com/blog/2011/02/create-a-pipe-cap.html) in
Revit 2011 over a year ago.

Now Mick Kulik of
[Hydratec, Inc.](http://www.hydracad.com) discovered
that the existing sample code did not work in his case, and worked together with Saikat Bhattacharya at the AutoCAD DevLab in San Rafael to fix this issue for Revit 2012.

Here is the original problem and its solution:

**Question:** Following the post about
[creating a pipe cap](http://thebuildingcoder.typepad.com/blog/2011/02/create-a-pipe-cap.html),
I was not able to get a cap to connect to a pipe in Revit MEP 2012.
The Error is "Line is too short".
Is there a new way to connect caps to pipes in Revit MEP 2012?

**Answer:** Saikat came down to give us a hand last week at the AutoCAD DevLab in San Rafael.
He had some sample code to attach caps to sloping pipes.
I was able to massage his code to fit our needs, and have attached his
[sample project CapAtStartOfPipe.zip](zip/CapAtStartOfPipe.zip) containing
the entire source code and Visual Studio solution implementing this command.

Saikat's code worked on all the cases we tested except when trying to connect the cap to a non-sloped vertical pipe, so I built in a case to handle it which worked for my test cases:
```csharp
  // rotate the cap if necessary

  // rotate about Z first
  XYZ pipeHorizontalDirection
    = new XYZ( dir.X, dir.Y, 0.0 ).Normalize();

  ConnectorSet c2
    = fi.MEPModel.ConnectorManager.Connectors;

  Connector capConnector = null;

  foreach( Connector c in c2 )
  {
    capConnector = c;
  }

  XYZ connectorDirection
    = -capConnector.CoordinateSystem.BasisZ;

  double zRotationAngle
    = pipeHorizontalDirection.AngleTo(
      connectorDirection );

  Transform trf = Transform.get\_Rotation(
    start, XYZ.BasisZ, zRotationAngle );

  XYZ testRotation = trf.OfVector(
    connectorDirection ).Normalize();

  if( Math.Abs( testRotation.DotProduct(
    pipeHorizontalDirection ) - 1 ) > 0.00001 )
  {
    zRotationAngle = -zRotationAngle;
  }

  Line axis = doc.Application.Create.NewLineBound(
    start, start + XYZ.BasisZ );

  ElementTransformUtils.RotateElement(
    doc, fi.Id, axis, zRotationAngle );

  // Need to rotate vertically?

  if( Math.Abs( dir.DotProduct( XYZ.BasisZ ) )
    > 0.000001 )
  {
    // if pipe is straight up and down,
    // kludge it my way else

    if( dir.X == 0 && dir.Y == 0 && dir.Z != 0 )
    {
      XYZ yaxis = new XYZ( 0.0, 1.0, 0.0 );

      double rotationAngle = dir.AngleTo( yaxis );

      if( dir.Z == 1 )
      {
        rotationAngle = -rotationAngle;
      }

      axis = doc.Application.Create.NewLine(
        start, yaxis, false );

      ElementTransformUtils.RotateElement(
        doc, fi.Id, axis, rotationAngle );
    }
    else
    {
      #region sloped pipes

      double rotationAngle = dir.AngleTo(
        pipeHorizontalDirection );

      XYZ normal = pipeHorizontalDirection
        .CrossProduct( XYZ.BasisZ );

      trf = Transform.get\_Rotation(
        start, normal, rotationAngle );

      testRotation = trf.OfVector( dir ).Normalize();

      if( Math.Abs( testRotation.DotProduct(
        pipeHorizontalDirection ) - 1 ) < 0.00001 )
      {
        rotationAngle = -rotationAngle;
      }

      axis = doc.Application.Create.NewLineBound(
        start, start + normal );

      ElementTransformUtils.RotateElement(
        doc, fi.Id, axis, rotationAngle );

      #endregion
    }
  }

  doc.Regenerate();

  // pick connector on cap:

  ConnectorSet connectors
    = fi.MEPModel.ConnectorManager.Connectors;

  Debug.Assert( 1 == connectors.Size,
    "expected only one connector on pipe cap element" );

  Connector cap\_end = null;

  foreach( Connector c in connectors )
  {
    cap\_end = c;
  }

  // pick closest connector on pipe:

  connectors = pipe.ConnectorManager.Connectors;

  Connector pipe\_end = null;
  double dist = double.MaxValue;
  foreach( Connector c in connectors )
  {
    XYZ p = c.Origin;
    double d = p.DistanceTo( start );
    if( d < dist )
    {
      dist = d;
      pipe\_end = c;
    }
  }
  pipe\_end.ConnectTo( cap\_end );

  doc.Regenerate();

  // After ConnectTo, the cap size is double, so
  // need to change the nominal radius to half.

  if( fi != null )
  {
    ConnectorSet pipeConnectors
      = pipe.ConnectorManager.Connectors;

    Connector pipeEnd = null;

    foreach( Connector c in pipeConnectors )
    {
      pipeEnd = c;
    }
    double radius = pipeEnd.Radius;

    Parameter para = fi.get\_Parameter(
      "Nominal Radius" );

    if( para != null && !para.IsReadOnly )
    {
      para.Set( radius );
    }
  }
```

Here is a screen capture of the result of testing caps on a vertical pipe:

![Cap test on vertical pipe](img/CapTest1.png)

Many thanks to Saikat and Mick for providing this new sample!