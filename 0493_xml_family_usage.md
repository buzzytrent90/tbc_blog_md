---
post_number: "0493"
title: "XML Family Usage Report"
slug: "xml_family_usage"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'python', 'revit-api', 'views']
source_file: "0493_xml_family_usage.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0493_xml_family_usage.html"
---

### XML Family Usage Report

I spent a pleasant weekend here in Tel Aviv, mostly preparing for the upcoming developer conferences.
I did take time for a walk along the beach to Jaffa and an early dinner in a neat little restaurant in the port,
[Container](http://www.container.org.il).
Here it is, behind the fishing boats, seen from the other side of the port, with old Jaffa in the background:

![Restaurant Container in Jaffa](img/restaurant_container_in_jaffa.jpg)

I also found out that you can get really good coffee in Israel, and wonderful ice cream, for instance in 'tita', and
in 'Anita, la Mamma del Gelato'.
Anita apparently won prizes for the best ice cream here three years running, which is quite something in view of the competition.
I really like the atmosphere of this city!

Today we move from one extreme to the next, from plus 25 degrees temperature here in Tel Aviv to a similar negative temperature in Moscow...

Meanwhile, here is another little item from Kevin Vandecar's
[filtering and optimisation](http://thebuildingcoder.typepad.com/blog/2010/10/filtered-element-collectors.html) class
at the AEC DevCamp in June, which I also used in my
[AU class CP234-2](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-university-2010-class-materials.html) on
the same topic.

As an example of using Revit filtering to retrieve family symbols and instances in conjunction with effective use of LINQ and a .NET framework XML library, Kevin presents an external command which creates a pretty comprehensive family usage report in very few lines of code.

It uses the ElementClassFilter, FamilySymbolFilter, and FamilyInstanceFilter classes to gather information about the families in a project.
For each family, it iterates over each symbol within the family, and finally retrieves all instances of each symbol.
These filters have not previously been discussed here.

This produces an abundance of information in a typical project, so it processes the results using the LINQ to XML functional or "DOM free" approach to produce a family inventory of the model in a nice XML format.

The report also includes the location of each family instance, and makes use of this helper method to return a string representing it.
The string consists of the endpoints of the given family instance location curve, if it has one, otherwise its location point, if it has one, or "<none>" if all else fails:
```csharp
static string LocationString( FamilyInstance fi )
{
  LocationPoint p = fi.Location as LocationPoint;
  LocationCurve c = fi.Location as LocationCurve;
  return ( null == p
    ? ( ( null == c
      ? "<none>"
      : Util.PointString( c.Curve.get\_EndPoint( 0 ) )
        + " to "
        + Util.PointString( c.Curve.get\_EndPoint( 1 ) ) ) )
    : Util.PointString( p.Point ) );
}
```

Here is the external command implementation, performing the following steps:

- Instantiate a stopwatch for benchmarking purposes.- Create a top level "Family\_Inventory" XML node.- Retrieve all families in the document and iterate over them.- For each family, create a family element node.- Retrieve all symbols using a FamilySymbolFilter, and iterate over those.- For each symbol, create a symbol node.- Retrieve all symbol instances in the active view using a FamilyInstanceFilter.- Create subnodes for all the instances.- Create and display the XML document and the elapsed time.

```python
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  UIApplication uiapp = commandData.Application;
  Document doc = uiapp.ActiveUIDocument.Document;

  Stopwatch sw = Stopwatch.StartNew();

  XElement xmlFamilyInstances = new XElement(
    "Family\_Inventory" );

  // retrieve all families.
  // use the ElementClassFilter shortcut
  // and filter all "Family" elements.

  FilteredElementCollector families
    = new FilteredElementCollector( doc );

  families.OfClass( typeof( Family ) );

  int nFamily = 0;
  int nSymbol = 0;
  int nInstance= 0;

  foreach( Family family in families )
  {
    ++nFamily;

    // XML: Start by adding the Family element

    XElement temp = new XElement(
      "FamilyName", family.Name );

    // use the FamilySymbolFilter for each Family

    FamilySymbolFilter filterFamSym
      = new FamilySymbolFilter( family.Id );

    FilteredElementCollector famSymbols
      = new FilteredElementCollector( doc );

    famSymbols.WherePasses( filterFamSym );

    foreach( FamilySymbol famSymbol in famSymbols )
    {
      ++nSymbol;

      FamilyInstanceFilter filterFamilyInst
        = new FamilyInstanceFilter(
          doc, famSymbol.Id );

      FilteredElementCollector collectorFamInstances
        = new FilteredElementCollector(
          doc, doc.ActiveView.Id );

      IEnumerable<FamilyInstance> famInstances
        = collectorFamInstances
          .WherePasses( filterFamilyInst )
          .OfType<FamilyInstance>();

      int nInstanceCount
        = famInstances.Count<FamilyInstance>();

      nInstance += nInstanceCount;

      temp.Add( new XElement(
        "SymbolName",
        famSymbol.Name,
        from fi in famInstances
          select new XElement(
            "Instance",
            fi.Id.ToString(),
            new XElement( "Type",
              fi.GetType().ToString() ),
            new XElement( "Position",
              LocationString( fi ) ) ) ) );
    }
    xmlFamilyInstances.Add( temp );
  }

  // Create the XML report document

  XDocument xmldoc =
    new XDocument(
        new XDeclaration( "1.0", "utf-8", "yes" ),
        new XComment(
          "Current Family Inventory of Revit project: "
          + doc.PathName ),
        xmlFamilyInstances );

  string fileName = "C:/FamilyInventory.xml";
  xmldoc.Save( fileName );

  Util.ShowElapsedTime( sw,
    "Linq Example 3 XML Report",
    string.Format( "{0} families with {1} symbols and {2} instances",
      nFamily, nSymbol, nInstance ),
    string.Empty );

  // We can use Internet Explorer or whatever
  // your favorite XML viewer is...
  Process.Start(
    "C:/Program Files/Internet Explorer/iexplore.exe",
    fileName);

  // Here is one that is free and is a little more
  // robust than Internet Explorer. If interested,
  // download from here:
  // http://download.cnet.com/XML-Marker/3000-7241\_4-10202365.html
  //Process.Start( @"C:/Program Files (x86)/XML Marker/xmlmarker.exe", fileName );

  return Result.Succeeded;
}
```

Running this in the ArchSample.rvt model produces the following elapsed time report:

```
Linq Example 3 XML Report:
729 milliseconds 57 families
with 61 symbols and 327 instances
```

Looking at the XML file in the browser, here are the completely collapsed contents showing just the top level family inventory node:

![Family inventory top level node](img/family_inventory_0.png)

Here are some opened family nodes, a few with no symbols defined, and the system panel family containing two symbols, one of which has no instances in the model:

![Family inventory family nodes](img/family_inventory_1.png)

Finally, here are some expanded instance nodes for one of the rectangular mullion symbols:

![Family inventory instance nodes](img/family_inventory_2.png)