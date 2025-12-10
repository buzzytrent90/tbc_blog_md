---
post_number: "0285"
title: "Rectangular Duct Corners"
slug: "rectangular_duct_corners"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'geometry', 'levels', 'revit-api', 'selection', 'walls', 'windows']
source_file: "0285_rectangular_duct_corners.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0285_rectangular_duct_corners.html"
---

### Rectangular Duct Corners

In between the series of background information from Scott's Autodesk University presentation on
[analysing building geometry](http://thebuildingcoder.typepad.com/blog/2010/01/analyse-building-geometry.html),
let's have a quick look at a practical application.

This is an MEP-specific issue on how to determine the corners of a rectangular duct.
One approach to determine this information is to perform the following steps:

- Analyse the duct geometry.- Extract its solid.- Iterate over the faces.- Select the face containing the rectangular connector.- Determine its four corners.

We explored the geometry of various elements in detail in the past, e.g.
[slab boundary](http://thebuildingcoder.typepad.com/blog/2008/10/slab-boundary.html),
[slab side faces](http://thebuildingcoder.typepad.com/blog/2008/11/slab-side-faces.html),
[wall elevation profile](http://thebuildingcoder.typepad.com/blog/2008/11/wall-elevation-profile.html),
[2D polygon areas and outer loop](http://thebuildingcoder.typepad.com/blog/2008/12/2d-polygon-areas-and-outer-loop.html),
[3D polygon areas](http://thebuildingcoder.typepad.com/blog/2008/12/3d-polygon-areas.html), and
[cylindrical columns](http://thebuildingcoder.typepad.com/blog/2009/04/cylindrical-column.html).
We have also had a look at
[MEP connectors](http://thebuildingcoder.typepad.com/blog/2009/04/mep-connectors.html) and
used the information contained in them, e.g. for the
[modeless pressure drop tool](http://thebuildingcoder.typepad.com/blog/2009/10/modeless-pressure-drop-tool.html).
This analysis now shows how these two sources of information, geometry and connectors, can be usefully combined.

The following solution comes from a case handled by Joe Ye and an implementation by Aaron and Vico in the Revit development team.

**Question:** I need to determine the four corners at the end of a rectangular duct.
Given a connector at the end of the duct, I have been using the connector coordinate system transform and width and height properties to determine the corners like this:
```csharp
  XYZ p = connector.CoordinateSystem.OfPoint(
    new XYZ( connector.Width / 2,
      connector.Height / 2, 0 ) );
```

This only seems to work if the duct is oriented such that the airflow is in the XY plane.

If the duct's airflow is along the Z axis so the duct is going up or down, I can calculate the corner like this:
```csharp
  XYZ p = connector.CoordinateSystem.OfPoint(
    new XYZ( connector.Height / 2,
      connector.Width / 2, 0 ) );
```

I seem to be missing something, because it seems like I shouldn't have to determine the duct's orientation.
Is there a more general solution to determining the corners of a rectangular duct?
Also, if the duct has been rotated, neither of these methods works.

**Answer:** There is no direct access to the four corners, but they can easily be determined from the duct geometry in combination with the connector location:

- Retrieve the duct geometry solid.- Retrieve the duct centre line and find the two rectangular end faces which are perpendicular to it.- Alternatively, find the planar rectangular face containing the connector.- Retrieve its four vertices to obtain the desired corner points.

I added a new external command CmdRectDuctCorners to The Building Coder sample application to demonstrate how these steps can be implemented.

This implementation also demonstrates making use of the .NET diagnostics Trace class to create a log file to write the data to, rather than using our customary approach of writing to the Visual Studio debug console window.
Using the Trace class and writing to a file is useful for driving automated tests and comparing the results with an expected template output file, for instance.

We make use of the following helper methods:

- GetFirstRectangularConnector – return the first rectangular connector of the given duct element.- FaceContainsConnector – return true if the given face of a solid contains the given connector.- AnalyseDuct – analyse the given duct element: determine its first rectangular connector, retrieve its solid, find the face containing the connector, and list its four vertices.

Here is the implementation of GetFirstRectangularConnector:
```csharp
static bool GetFirstRectangularConnector(
  Duct duct,
  out Connector c1 )
{
  c1 = null;

  ConnectorSet connectors
    = duct.ConnectorManager.Connectors;

  if( 0 < connectors.Size )
  {
    foreach( Connector c in connectors )
    {
      if( ConnectorProfileType.RectProfile
        == c.Shape )
      {
        c1 = c;
        break;
      }
      else
      {
        Trace.WriteLine( "Connector shape: "
          + c.Shape );
      }
    }
  }
  return null != c1;
}
```

FaceContainsConnector is short and sweet.
It uses Face.Project to determine the closest point on the face to the connector origin.
If it exists and the distance returned by the method is small, the connector is contained in the face:
```csharp
static bool FaceContainsConnector(
  Face face,
  Connector c )
{
  XYZ p = c.Origin;

  IntersectionResult result = face.Project( p );

  return null != result
    && Math.Abs( result.Distance ) < 1e-9;
}
```

The mainline of the command does the following:

- Check that we really are running in Revit MEP. If that is not the case, we will not have access to the duct element's connector manager.- Set up the log file for the Trace mechanism.
    Its path and filename is determined from the location and name of the executing assembly with a suffix of the current date and an extension ".log".- Set up an error handler and manage exceptions, including final cleanup to ensure the trace mechanism is properly closed.- Iterate over the selected elements, check that each one really is a duct, and pass it into the analysis method.- Return Failed regardless of whether the analysis was successful or not, so that the database is not unnecessarily
          [marked as dirty](http://thebuildingcoder.typepad.com/blog/2008/12/document-ismodified-property.html)
          even though nothing was modified.

```csharp
Application app = commandData.Application;

if( ProductType.MEP != app.Product )
{
  message = "Please run this command in Revit MEP.";
  return CmdResult.Failed;
}

Document doc = app.ActiveDocument;
SelElementSet sel = doc.Selection.Elements;

if( 0 == sel.Size )
{
  message = "Please select some rectangular ducts.";
  return CmdResult.Failed;
}

// set up log file:

string log = Assembly.GetExecutingAssembly().Location
  + "." + DateTime.Now.ToString( "yyyyMMdd" )
  + ".log";

if( File.Exists( log ) )
{
  File.Delete( log );
}

TraceListener listener
  = new TextWriterTraceListener( log );

Trace.Listeners.Add( listener );

try
{
  Trace.WriteLine( "Begin" );

  // loop over all selected ducts:

  foreach( Duct duct in sel )
  {
    if( null == duct )
    {
      Trace.TraceError( "The selection is not a duct!" );
    }
    else
    {
      // process each duct:

      Trace.WriteLine( "========================" );
      Trace.WriteLine( "Duct: Id = " + duct.Id.Value );

      AnalyseDuct( duct );
    }
  }
}
catch( Exception ex )
{
  Trace.WriteLine( ex.ToString() );
}
finally
{
  Trace.Flush();
  listener.Close();
  Trace.Close();
  Trace.Listeners.Remove( listener );
}
return CmdResult.Failed;
```

Finally, here is the AnalyseDuct method that does the real work:

- Retrieve the duct's first rectangular connector.- Retrieve the duct geometry.- Extract the solid.- Iterate over its faces.- Find the face containing the connector.- List its four vertices to the log file.

```csharp
static bool AnalyseDuct( Duct duct )
{
  bool rc = false;

  Connector c1;
  if( !GetFirstRectangularConnector( duct, out c1 ) )
  {
    Trace.TraceError( "The duct is not rectangular!" );
  }
  else
  {
    Options opt = new Options();
    opt.DetailLevel = Options.DetailLevels.Fine;
    GeoElement geoElement = duct.get\_Geometry( opt );

    foreach( GeometryObject obj in geoElement.Objects )
    {
      Solid solid = obj as Solid;
      if( solid != null )
      {
        bool foundFace = false;
        foreach( Face face in solid.Faces )
        {
          foundFace = FaceContainsConnector( face, c1 );
          if( foundFace )
          {
            Trace.WriteLine( "==> Four face corners:" );

            EdgeArray a = face.EdgeLoops.get\_Item( 0 );

            foreach( Edge e in a )
            {
              XYZ p = e.Evaluate( 0.0 );

              Trace.WriteLine( "Point = "
                + Util.PointString( p ) );
            }
            rc = true;
            break;
          }
        }
        if( !foundFace )
        {
          Trace.WriteLine( "[Error] Face not found" );
        }
      }
    }
  }
  return rc;
}
```

This is the resulting log file output after running this command with a simple rectangular duct selected:

```
Begin
========================
Duct: Id = 431526
==> Four face corners:
Point = (-18.27,4.66,8.53)
Point = (-18.27,4.66,9.51)
Point = (-18.27,3.67,9.51)
Point = (-18.27,3.67,8.53)
```

Here is
[version 1.1.0.58](zip/bc11058.zip)
of the complete Building Coder sample source code and Visual Studio solution including the new command.

Many thanks to Joe for handling this case and to Aaron and Vico for the initial implementation!