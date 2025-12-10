---
post_number: "0010"
title: "Selecting all Walls"
slug: "selecting_all_walls"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'filtering', 'geometry', 'parameters', 'revit-api', 'views', 'walls', 'windows']
source_file: "0010_selecting_all_walls.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0010_selecting_all_walls.html"
---

### Selecting all Walls

Now that we have covered the basics, let us get into some small examples of day to day issues that arise. I recently gave a Revit API introductory workshop. The first little sample add-in we wrote together as a first step to exploring API access to the Revit BIM selects all wall elements in the database and lists the area and length of each. This provides an opportunity to discuss a number of interesting details, such as some of the debugging support included in the .NET framework, the handling of units in Revit, parameter access and location definition on Revit database elements, etc.

Let us discuss some elements of interest in the code before actually presenting it.

One .NET framework feature that I suggest exploring is the Debug namespace. We use it below to comfortably print out information to the Visual Studio debug output window using Debug.WriteLine(). Of course, this is only useful in a debugging context, but then it is much more comfortable than formatting stuff into a string for presentation in a message box, in which the user has the onerous task of clicking OK, and which is less handy to search for text in or copy and paste from.

I make use of 'using' statements to minimise code bloat and make the code as readable as possible. I also make use of aliases to define handy short cuts such as CmdResult instead of the rather hefty IExternalCommand.Result. In the code below, I avoid a statement using the entire Autodesk.Revit.Geometry namespace, since that would create an ambiguity between Autodesk.Revit.Element and Autodesk.Revit.Geometry.Element. Instead, I just create an alias for Autodesk.Revit.Geometry.XYZ and avoid pulling in all the rest of that namespace.

The code below demonstrates a simple use of the Revit 2009 filtered element access to access all wall elements in the building model, which is vastly faster than the element iteration in 2008.

We extract the wall area from the built-in parameter HOST\_AREA\_COMPUTED. Further down, you will note that this produced slightly unexpected results, so we also added code to check each wall length by retrieving the start and end point of its location curve.

Here is the complete code of this little external command:

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit;
using Autodesk.Revit.Elements;
using Autodesk.Revit.Parameters;
using CmdResult = Autodesk.Revit.IExternalCommand.Result;
using XYZ = Autodesk.Revit.Geometry.XYZ;
namespace SelectWalls
{
  public class Util
  {
    static public string RealString( double a )
    {
      return a.ToString( "0.##" );
    }
  }

  public class Command : IExternalCommand
  {
    public CmdResult Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      Application app = commandData.Application;
      Document doc = app.ActiveDocument;
      Type t = typeof( Wall );
      Filter f = app.Create.Filter.NewTypeFilter( t );
      List<Element> walls = new List<Element>();
      int n = doc.get\_Elements( f, walls );
      foreach( Wall wall in walls )
      {
        Parameter param = wall.get\_Parameter(
          BuiltInParameter.HOST\_AREA\_COMPUTED );
        double a = ( ( null != param )
          && (StorageType.Double == param.StorageType) )
          ? param.AsDouble()
          : 0.0;
        string s = ( null != param )
          ? param.AsValueString()
          : "null";
        LocationCurve lc = wall.Location as LocationCurve;
        XYZ p = lc.Curve.get\_EndPoint( 0 );
        XYZ q = lc.Curve.get\_EndPoint( 1 );
        double l = q.Distance( p );
        string format
          = "Wall <{0} {1}> length {2} area {3} ({4})";
        Debug.WriteLine( string.Format( format,
          wall.Id.Value.ToString(), wall.Name,
          Util.RealString( l ), Util.RealString( a ),
          s ) );
      }
      return CmdResult.Succeeded;
    }
  }
}
```

We need some walls to run this on. As a sample model, we can use the little house created by the Lab2\_0\_CreateLittleHouse command in the code accompanying the Revit API introduction, which looks like this in plan view:

![Little House Plan View](img/LittleHouse01.png)

Here is the 3D view of the sample model:

![Little House 3D View](img/LittleHouse02.png)

Running the command in this sample produces the following in the debug output window:

```
Wall <127145 Generic - 200mm> length 22.97 area 283.65 (26.352 m²)
Wall <127146 Generic - 200mm> length 13.12 area 172.22 (16.000 m²)
Wall <127147 Generic - 200mm> length 22.97 area 301.39 (28.000 m²)
Wall <127148 Generic - 200mm> length 13.12 area 163.61 (15.200 m²)
```

There are various items of interest here. One of them stems from the fact when dealing with length and area, we need to consider the units being used. This will be the topic of a future posting.

Another question is this: Why do we see four different wall areas in the list? The little house is rectangular, so we would expect the walls and thus their areas to be pairwise identical. In fact, this is what prompted us to list the location line length as well, where we note that these are indeed pairwise identical. The reason has to do with the Revit product and the way it handles the wall joins at the corners. If we set up all the walls to connect to their neighbours using a mitered wall join, then the reported areas will be identical for walls of equal length.