---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.5
content_type: qa
optimization_date: '2025-12-11T11:44:13.234797'
original_url: https://thebuildingcoder.typepad.com/blog/0016_relationship_inverter.html
post_number: '0016'
reading_time_minutes: 6
series: general
slug: relationship_inverter
source_file: 0016_relationship_inverter.htm
tags:
- csharp
- doors
- elements
- family
- filtering
- geometry
- parameters
- revit-api
- walls
- windows
title: Relationship Inverter
word_count: 1169
---

### Relationship Inverter

Moving away from the topic of geometry, I have another sample that I would like to share from the Revit API introductory workshop that I recently held. In this sample, we address the following topics:

- Using a Boolean combination of Revit API filters to find certain elements.
- Retrieving the hosted to host relationship between doors, windows, and walls.
- Inverting the relationship, i.e. determining the host to hosted one.

This sample also introduces some new little utility functions, such as ElementDescription() to return a string describing a given Revit element.

As said, we are interested in the relationship between door and window openings and the walls hosting them. The Revit API provides a one-way relationship from hosted element to its host in the form of the FamilyInstance.Host property, which returns the containing element if the family instance is contained within another element. In our case, the doors and windows are family instances, and the containing element is a wall.

In a first step, we create a Boolean combination of filters to select the elements we are interested in from the building model database. We do not need to retrieve the walls, because they do not provide any relationship information about their hosted elements. All we need is to access the doors and windows. These can be identified by having the class FamilyInstance and either door or window category. We can use a TypeFilter to match the class, and CategoryFilter instances for the categories. The category filters can be combined using a Boolean OR to match either door or window. The resulting filter can in turn be combined with the class filter using a Boolean AND. The result is described in the code below as

```
f5 = f1 && f4
   = f1 && (f2 || f3)
   = family instance and (door or window)
```

Once all doors and windows in the model have been retrieved, we can simply loop through them and query the host of each. For each host element id, we create a new entry into a dictionary using the host element id as a key and a list of all hosted element ids as its value. The elements being processed are also logged to the Visual Studio debug output window using the Debug.WriteLine() method provided by the very useful System.Diagnostics namespace.

At this point, we are actually already done. The resulting dictionary now implements the inverted relationship. We started out with each hosted element knowing its host, and now we have the dictionary with the host element id as key, and a list of ids of all its hosted elements as a value. All that remains to do is list this information in the debug output window, or use it in some other way.

Here is the code for this little command:

```csharp
#region Namespaces
using System;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit;
using Autodesk.Revit.Elements;
using Autodesk.Revit.Parameters;
using CmdResult
  = Autodesk.Revit.IExternalCommand.Result;
#endregion // Namespaces

namespace BuildingCoder
{
  public class CmdRelationshipInverter
    : IExternalCommand
  {
    private Document m\_doc;

    public static string PluralSuffix( int n )
    {
      return 1 == n ? "" : "s";
    }

    public static string DotOrColon( int n )
    {
      return 1 < n ? ":" : ".";
    }

    string ElementDescription( Element e )
    {
      // for a wall, the element name equals the
      // wall type name, which is equivalent to the
      // family name ...
      FamilyInstance fi = e as FamilyInstance;
      string fn = string.Empty;
      if( null != fi )
      {
        fn = fi.Symbol.Family.Name + " ";
      }
      return string.Format( "{0} {1}<{2} {3}>",
        e.Category.Name, fn,
        e.Id.Value.ToString(), e.Name );
    }

    string ElementDescription( ElementId id )
    {
      Element e = m\_doc.get\_Element( ref id );
      return ElementDescription( e );
    }

    private Dictionary<ElementId, List<ElementId>>
      getElementIds( List<Element> elements )
    {
      Dictionary<ElementId, List<ElementId>> dict =
        new Dictionary<ElementId, List<ElementId>>();

      string fmt = "{0} is hosted by {1}";

      foreach( FamilyInstance fi in elements )
      {
        ElementId id = fi.Id;
        ElementId idHost = fi.Host.Id;

        Debug.WriteLine( string.Format( fmt,
          ElementDescription( fi ),
          ElementDescription( idHost ) ) );

        if( !dict.ContainsKey( idHost ) )
        {
          dict.Add( idHost, new List<ElementId>() );
        }
        dict[idHost].Add( id );
      }
      return dict;
    }

    private void dumpHostedElements(
      Dictionary<ElementId, List<ElementId>> ids )
    {
      foreach( ElementId idHost in ids.Keys )
      {
        string s = string.Empty;

        foreach( ElementId id in ids[idHost] )
        {
          if( 0 < s.Length )
          {
            s += ", ";
          }
          s += ElementDescription( id );
        }

        int n = ids[idHost].Count;

        Debug.WriteLine(string.Format(
          "{0} hosts {1} opening{2}: {3}",
          ElementDescription( idHost ),
          n, PluralSuffix( n ), s ) );
      }
    }

    public CmdResult Execute(
      ExternalCommandData commandData,
      ref string message,
      ElementSet elements )
    {
      Application app = commandData.Application;
      m\_doc = app.ActiveDocument;

      // f5 = f1 && f4
      //    = f1 && (f2 || f3)
      //    = family instance and (door or window)
      Autodesk.Revit.Creation.Filter cf
        = app.Create.Filter;

      Filter f1 = cf.NewTypeFilter(
        typeof( FamilyInstance ) );

      Filter f2 = cf.NewCategoryFilter(
        BuiltInCategory.OST\_Doors );
      Filter f3 = cf.NewCategoryFilter(
        BuiltInCategory.OST\_Windows );

      Filter f4 = cf.NewLogicOrFilter( f2, f3 );
      Filter f5 = cf.NewLogicAndFilter( f1, f4 );

      List<Element> openings = new List<Element>();
      int n = m\_doc.get\_Elements( f5, openings );

      // map with key = host element id and
      // value = list of hosted element ids:
      Dictionary<ElementId, List<ElementId>> ids =
        getElementIds( openings );

      dumpHostedElements( ids );
      m\_doc = null;

      return CmdResult.Succeeded;
    }
  }
}
```

Here is a log file from a sample run. As you can see, we first traverse all the hosted elements and determine the hosting wall of each. These are inserted into the dictionary representing the inverse relationship, which dumps its contents using dumpHostedElements() before the command ends. Please excuse the long overflowing lines:

```
Doors M_Single-Flush <127252 0915 x 2134mm> is hosted by Walls <127248 Generic - 200mm>
Windows M_Fixed <127255 0406 x 0610mm> is hosted by Walls <127248 Generic - 200mm>
Windows M_Fixed <127258 0406 x 0610mm> is hosted by Walls <127248 Generic - 200mm>
Doors M_Single-Flush <127295 0915 x 2134mm> is hosted by Walls <127167 Generic - 200mm>
Doors M_Single-Flush <127331 0915 x 2134mm> is hosted by Walls <127167 Generic - 200mm>
Windows M_Fixed <127356 0406 x 0610mm> is hosted by Walls <127240 Generic - 200mm>
Windows M_Fixed <127411 0406 x 0610mm> is hosted by Walls <127240 Generic - 200mm>
Windows M_Fixed <127436 0406 x 0610mm> is hosted by Walls <127240 Generic - 200mm>
Windows M_Fixed <127462 0406 x 0610mm> is hosted by Walls <127215 Generic - 200mm>
Windows M_Fixed <127484 0406 x 0610mm> is hosted by Walls <127215 Generic - 200mm>
Windows M_Fixed <127502 0406 x 0610mm> is hosted by Walls <127215 Generic - 200mm>
Windows M_Fixed <127526 0406 x 0610mm> is hosted by Walls <127215 Generic - 200mm>
Walls <127248 Generic - 200mm> hosts 3 openings: Doors M_Single-Flush <127252 0915 x 2134mm>, Windows M_Fixed <127255 0406 x 0610mm>, Windows M_Fixed <127258 0406 x 0610mm>
Walls <127167 Generic - 200mm> hosts 2 openings: Doors M_Single-Flush <127295 0915 x 2134mm>, Doors M_Single-Flush <127331 0915 x 2134mm>
Walls <127240 Generic - 200mm> hosts 3 openings: Windows M_Fixed <127356 0406 x 0610mm>, Windows M_Fixed <127411 0406 x 0610mm>, Windows M_Fixed <127436 0406 x 0610mm>
Walls <127215 Generic - 200mm> hosts 4 openings: Windows M_Fixed <127462 0406 x 0610mm>, Windows M_Fixed <127484 0406 x 0610mm>, Windows M_Fixed <127502 0406 x 0610mm>, Windows M_Fixed <127526 0406 x 0610mm>
```

For your convenience, I am adding the complete Visual Studio solution here. This version 1.0.0.3 includes the three commands we discussed so far: CmdListWalls, CmdRelationshipInverter and CmdWallDimensions.