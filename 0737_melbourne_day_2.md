---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 5.2
content_type: code_example
optimization_date: '2025-12-11T11:44:14.488337'
original_url: https://thebuildingcoder.typepad.com/blog/0737_melbourne_day_2.html
post_number: '0737'
reading_time_minutes: 7
series: general
slug: melbourne_day_2
source_file: 0737_melbourne_day_2.htm
tags:
- csharp
- doors
- elements
- filtering
- geometry
- python
- references
- revit-api
- selection
- transactions
- views
- walls
- windows
title: Melbourne Day Two
word_count: 1438
---

﻿

### Melbourne Day Two

On the second day of the Revit API training here in Melbourne, we addressed a number of further interesting issues, both basic and beyond.
Our sample code ended up demonstrating the following functionality:

- Revit MEP model creation: [place a duct](#1).- Geometrical analysis: [retrieving unique geometry vertices](#2) from a selected element.

Besides that, we looked at
[extensible storage](#3),
and I compiled an updated version of the
[RstLink sample for Revit Structure 2012](#4).

I also received interesting comments on a previous post on finding the
[Revit parent window](#5) that
is worthwhile sharing here.

After the training, I went for a quick climb with Rob at
[Hardrock](http://www.hardrock.com.au),
marvelling at yet another grading system.
![Hardrock climbing](img/hardrock.jpg)

Here in Australia I went for the easy grades 16 to 18.
No idea what they compare to in European or American grading systems.

After that, I had a sandwich in the
[Siglo](http://www.theage.com.au/news/bar-reviews/siglo-bar/2008/05/12/1210444319156.html) roof-top
bar on 161 Spring Street above
the
[Melbourne Supper Club](http://www.theage.com.au/news/bar-reviews/the-melbourne-supper-club/2006/04/03/1143916451849.html) in
the same house.
These two places share an extremely sympathetic anonymous entrance, a simple brown wooden door with no sign whatsoever.
If no-one told you they are there, you would never find them.

#### Placing a Duct

One thing we talked about was creating new elements in the model.

As a very simple example, we created the code to place an HVAC duct:
```python
[Transaction( TransactionMode.Manual )]
public class Command3 : IExternalCommand
{
  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    XYZ p, q;

    try
    {
      Selection sel = uidoc.Selection;
      p = sel.PickPoint( "Start point: " );
      q = sel.PickPoint( "End point: " );
    }
    catch( RvtOperationCanceledException )
    {
      return Result.Cancelled;
    }

    DuctType ductType
      = new FilteredElementCollector( doc )
        .OfClass( typeof( DuctType ) )
        .Cast<DuctType>()
        .FirstOrDefault<DuctType>();

    if( null == ductType )
    {
      message = "No duct type found.";
      return Result.Failed;
    }

    //Duct duct = new Duct(); // OO approach, not supported
    //Duct duct = Duct.Create( doc );

    Transaction tx = new Transaction( doc );
    tx.Start( "Add Duct" );

    Duct duct = doc.Create.NewDuct( p, q, ductType );

    tx.Commit();

    return Result.Succeeded;
  }
}
```

We also played with creating a mechanical system.
That needs some fittings a well, because their connectors provide the required duct system type information.

Look at the AutoRoute and AvoidObstruction SDK samples for more details.

#### Retrieving Unique Geometry Vertices

We then explored geometry extraction, and especially the identification of unique points with in it, since this bring up a number of issues regarding units and precision etc.

Here is a simple command that prompts me to select an element and accesses its geometry.
It processes it as well towards the end; ignore that for the moment:
```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  UIDocument uidoc = uiapp.ActiveUIDocument;
  Application app = uiapp.Application;
  Document doc = uidoc.Document;

  Element e;

  try
  {
    Reference r = uidoc.Selection.PickObject(
      ObjectType.Element,
      "Please pick an element." );

    e = doc.get\_Element( r.ElementId );
  }
  catch( RvtOperationCanceledException )
  {
    return Result.Cancelled;
  }

  Options opt = app.Create.NewGeometryOptions();

  GeometryElement geo = e.get\_Geometry( opt );

  Solid solid = null;

  foreach( GeometryObject obj in geo.Objects )
  {
    GeometryInstance inst = obj as GeometryInstance;

    if( null != inst )
    {
      geo = inst.GetSymbolGeometry();
      break;
    }
  }

  foreach( GeometryObject obj in geo.Objects )
  {
    solid = obj as Solid;

    if( null != solid )
    {
      break;
    }
  }

  if( null == solid )
  {
    message = "Unable to access element solid.";
    return Result.Failed;
  }
  Dictionary<XYZ, int> corners = GetCorners( solid );

  int n = corners.Count;

  Debug.Print( "{0} corners found:", n );

  foreach( XYZ p in corners.Keys )
  {
    Debug.Print( PointString( p ) );
  }

  return Result.Succeeded;
}
```

The next step we looked at is identification of all unique vertices in the geometry.
For this, we need to somehow implement a fuzzy method to distinguish between points that really are different, but detect that points that are nearly the same should in fact be treated as identical.
We can achieve this by implementing an XYZ equality comparer, e.g. like this:
```python
class XyzEqualityComparer : IEqualityComparer<XYZ>
{
  const double \_sixteenthInchInFeet
    = 1.0 / ( 16.0 \* 12.0 );

  public bool Equals( XYZ p, XYZ q )
  {
    return p.IsAlmostEqualTo( q,
      \_sixteenthInchInFeet );
  }

  public int GetHashCode( XYZ p )
  {
    return PointString( p ).GetHashCode();
  }
}
```

As an almost-equal tolerance we are using a rather large value, because Revit uses this pretty rough number in several places itself, e.g. to limit the limit the
[minimum line length](http://thebuildingcoder.typepad.com/blog/2009/07/think-big-in-revit.html).

You must be careful to also define a hash code that does not return different values for points that you wish to compare equal, or every single point will be considered different, possibly even when a point is compared with itself.
In this case we are using our two-decimal string representation to define a hash code, so many points that are considered different by the comparison operator will actually generate the same hash code.
In a perfect world, the hash code generator would be a bit more appropriately chosen to match the equality comparison precision.

We can easily implement a short and sweet method to extract all the unique corner vertices of the solid geometry by using the vertices themselves as keys in a dictionary based on our equality comparer like this:
```csharp
Dictionary<XYZ,int> GetCorners( Solid solid )
{
  Dictionary<XYZ, int> corners
    = new Dictionary<XYZ, int>(
      new XyzEqualityComparer() );

  foreach( Face f in solid.Faces )
  {
    foreach( EdgeArray ea in f.EdgeLoops )
    {
      foreach( Edge e in ea )
      {
        XYZ p = e.AsCurveFollowingFace( f )
          .get\_EndPoint( 0 );

        if( !corners.ContainsKey( p ) )
        {
          corners[p] = 0;
        }
        ++corners[p];
      }
    }
  }
  return corners;
}
```

The dictionary returned includes a count of how many times each vertex was encountered.

This is something very important that I have looked at numerous times in the past.
I already presented the fundamentals for handling this when looking at
[nested instance geometry](http://thebuildingcoder.typepad.com/blog/2009/05/nested-instance-geometry.html),
which also implements a GetVertices method similar to the above,
and reused it for
[toposurface point classification](http://thebuildingcoder.typepad.com/blog/2011/03/toposurface-interior-and-boundary-points.html) and
accessing the
[top faces](http://thebuildingcoder.typepad.com/blog/2011/07/top-faces-of-wall.html) of
a sloped wall.

#### Extensible Storage

Here is the documentation and sample code from my Autodesk University 2011
[class](http://au.autodesk.com/?nd=event_class&session_id=9263&jid=1725932) and
[lab](http://au.autodesk.com/?nd=event_class&session_id=9726&jid=1725932) on
this topic, which says it all:

- [CP4451\_tammik\_estorage.pdf](file:///C:/a/doc/revit/au/2011/doc/CP4451_tammik_estorage.pdf): handout document- [CP4451\_tammik\_estorage.zip](file:///C:/a/doc/revit/au/2011/doc/CP4451_tammik_estorage.zip): sample source code- [CP6760-L\_tammik\_estorage.pdf](file:///C:/a/doc/revit/au/2011/doc/CP6760-L_tammik_estorage.pdf): handout document- [CP6760-L\_tammik\_estorage\_lab.zip](file:///C:/a/doc/revit/au/2011/doc/CP6760-L_tammik_estorage_lab.zip) exercises

#### Revit Structure Link 2012

Long overdue, I finally updated the RstLink sample for Revit 2012 and AutoCAD 2012.
I provided a
[brief description of the analytical link sample](http://thebuildingcoder.typepad.com/blog/2009/04/revit-structure-resources.html) two
years ago, and updated the sample last year for
[Revit Structure 2011](http://thebuildingcoder.typepad.com/blog/2011/03/structural-analysis-links-to-revit-structure.html).
Now here is
[RstLink2012.zip](zip/RstLink2012.zip) containing the entire Visual Studio solution for the following projects:

- RstLink: helper DLL shared by both AutoCAD and Revit client.- RstLinkRevitClient: Revit command implementations: RSLinkImport, RSLinkExport, RsLinkLiveLink.- RstLinkAcadClient: AutoCAD command implementations: RSImport, RSExport, RSMakeMember.- RstLinkAcadClientDynProps: Dynamic Revit OPM properties for AutoCAD objects.

In fact, most of these are provided for both C# and VB, so it ends up including the following projects:

- RstLinkCs- RstLinkVb- RstLinkRevitClientCs- RstLinkRevitClientVb- RstLinkAcadClientCs- RstLinkAcadClientVb- Miro: a C++ project to generate the RstLinkAcadClientDynProps ARX application

#### Revit Parent Window

Finally, to wrap up this information overflow, Victor Chekalin, or Виктор Чекалин, offered a very useful and interesting
[suggestion](http://thebuildingcoder.typepad.com/blog/2010/06/revit-parent-window.html?cid=6a00e553e1689788330163030f0ea6970d#comment-6a00e553e1689788330163030f0ea6970d) for
a new and easier way to obtain the
[Revit parent window](http://thebuildingcoder.typepad.com/blog/2010/06/revit-parent-window.html):

1. Add a reference to the AdWindows assembly, located at the same place as the RevitAPI assembly, and set its Copy Local property to false.- Get the Revit main window handle using the static property ApplicationWindow of the ComponentManager class in the Autodesk.Windows namespace.

Now, you can use only one Property – ComponentManager.ApplicationWindow – instead two methods – get current process and main window handle.