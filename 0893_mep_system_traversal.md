---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.5
content_type: qa
optimization_date: '2025-12-11T11:44:14.811331'
original_url: https://thebuildingcoder.typepad.com/blog/0893_mep_system_traversal.html
post_number: 0893
reading_time_minutes: 3
series: mep
slug: mep_system_traversal
source_file: 0893_mep_system_traversal.htm
tags:
- csharp
- elements
- filtering
- revit-api
- transactions
- mep
title: Simple MEP System Traversal
word_count: 574
---

### Simple MEP System Traversal

Here is a simple MEP system traversal implementation that especially addresses the issue of determining what equipment is connected to the systems.

The following read-only external command traverses all MEP systems in the document, using the MEPSystem.Elements property to retrieve most of the desired elements with very little effort.

More than half the code is actually fussing about with formatting the result:
```csharp
[Transaction( TransactionMode.ReadOnly )]
public class Command : IExternalCommand
{
  static string PluralSuffix( int n )
  {
    return 1 == n ? "" : "s";
  }

  void TraverseSystems( Document doc )
  {
    FilteredElementCollector systems
      = new FilteredElementCollector( doc )
        .OfClass( typeof( MEPSystem ) );

    int i, n;
    string s;
    string[] a;

    StringBuilder message = new StringBuilder();

    foreach( MEPSystem system in systems )
    {
      message.AppendLine( "System Name: "
        + system.Name );

      message.AppendLine( "Base Equipment: "
        + system.BaseEquipment );

      ConnectorSet cs = system.ConnectorManager
        .Connectors;

      i = 0;
      n = cs.Size;
      a = new string[n];

      s = string.Format(
        "{0} element{1} in ConnectorManager: ",
        n, PluralSuffix( n ) );

      foreach( Connector c in cs )
      {
        Element e = c.Owner;

        if( null != e )
        {
          a[i++] = e.GetType().Name
            + " " + e.Id.ToString();
        }
      }

      message.AppendLine( s
        + string.Join( ", ", a ) );

      i = 0;
      n = system.Elements.Size;
      a = new string[n];

      s = string.Format(
        "{0} element{1} in System: ",
        n, PluralSuffix( n ) );

      foreach( Element e in system.Elements )
      {
        a[i++] = e.GetType().Name
          + " " + e.Id.ToString();
      }

      message.AppendLine( s
        + string.Join( ", ", a ) );
    }

    n = systems.Count<Element>();

    string caption =
      string.Format( "Traverse {0} MEP System{1}",
      n, (1 == n ? "" : "s") );

    TaskDialog dlg = new TaskDialog( caption );
    dlg.MainContent = message.ToString();
    dlg.Show();
  }

  public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements )
  {
    UIApplication uiapp = commandData.Application;
    UIDocument uidoc = uiapp.ActiveUIDocument;
    Document doc = uidoc.Document;

    TraverseSystems( doc );

    return Result.Succeeded;
  }
}
```

I executed this on an absolutely trivial system:

![Simple system](img/traverse_system_system.png)

That generates the following message box as a result:

![Resulting message](img/traverse_system_msg.png)

Obviously, you will want to clean up the reporting significantly to suit your needs.
Currently, the owner elements in the ConnectorSet are listed, and the same elements also appear in the system elements list.
Here is another result of running this on two systems, an electrical and a duct system, with the elements highlighted in yellow:

![Duplicated elements](img/traverse_system_duplicates.png)

Also, the API reports four elements in the system including one duct segment, whereas the UI reports three, which is what one would expect, so some identification of duplicate elements needs to be added to make this useful.

Furthermore, reporting on these ‘logical’ systems may have the limitation that in-line equipment such as duct dampers and valves are not reported as part of the system.
The other system traversal samples based on physical connectivity provide this info, however.

For now, the main point is to demonstrate that this simple access exists at all.

For your convenience, here is
[MepSystemTraversal.zip](zip/MepSystemTraversal.zip) containing
the source code, Visual Studio solution and add-in manifest for this command.

For more advanced traversal algorithms and determining the correct order of the individual system elements in the direction of the flow, you can look at the
[TraverseSystem SDK sample](http://thebuildingcoder.typepad.com/blog/2009/06/revit-mep-api.html)
([2010](http://thebuildingcoder.typepad.com/blog/2009/09/the-revit-mep-api.html#6),
[2011](http://thebuildingcoder.typepad.com/blog/2010/05/the-revit-mep-2011-api.html#samples))
for mechanical systems and the
[AdnRme](http://thebuildingcoder.typepad.com/blog/2012/05/the-adn-mep-sample-adnrme-for-revit-mep-2013.html) sample
for electrical ones.