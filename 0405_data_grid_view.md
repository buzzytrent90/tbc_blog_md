---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.4
content_type: code_example
optimization_date: '2025-12-11T11:44:13.895376'
original_url: https://thebuildingcoder.typepad.com/blog/0405_data_grid_view.html
post_number: '0405'
reading_time_minutes: 10
series: views
slug: data_grid_view
source_file: 0405_data_grid_view.htm
tags:
- csharp
- elements
- family
- filtering
- python
- revit-api
- views
- windows
title: Populating a Data Grid View
word_count: 2010
---

### Populating a Data Grid View

By the time you read this, I will be gone on the first of a series of holidays this summer.
By the way, for that reason, don't expect any answers to comments for a while.
I left the computer at home this time!
I posted this in advance to ensure you have something worthwhile to chew on during my absence.

This is another stepping stone towards implementing Martin Schmid's proposal to display and navigate through all unconnected MEP connectors in the model that I mentioned in the discussion on
[retrieving MEP elements and connectors](http://thebuildingcoder.typepad.com/blog/2010/06/retrieve-mep-elements-and-connectors.html).

Another one of the steps is
[determining the Revit parent window](http://thebuildingcoder.typepad.com/blog/2010/06/revit-parent-window.html) and
attaching a form to it.
The latter enables us to set Revit as the parent window of the modeless form, ensuring that it stays on top of Revit when Revit is visible, and also that it is minimised when the user minimises Revit.

After we have determined the unconnected connectors, we populate a modeless form with their pertinent information, display it, and enable the user to double click on an element to zoom to it in Revit.
The double click handling and zooming interaction still needs to be documented.
For the moment, we will focus on implementing, populating and displaying the modeless form.

#### Modeless Form with Data Grid View

I implemented a modeless form named LooseConnectorNavigator.
It is derived from a Form base class from the System.Windows.Forms namespace.
It has one single DataGridView control added to it which is anchored to all four sides.
Here is what it looks like with its properties in the Visual Studio form designer:

![Modeless loose connector navigator form](img/modeless_loose_connector_navigator_form.png)

A data grid view is extremely easy to populate with an absolute minimum of coding, as we have already seen in Joel Karr's sample showing how to
[list linked elements](http://thebuildingcoder.typepad.com/blog/2009/02/list-linked-elements.html).
If you set up an appropriate data container whose elements expose certain public member properties, all you need to do is to specify the container as the data source of the data grid view, and it will not only automatically populate the rows of data, but even create the columns according to the available properties!

What's more, by using
[auto-implemented properties](http://thebuildingcoder.typepad.com/blog/2009/07/three-coding-and-performance-hints.html),
the definition of the properties on the data item classes is reduced to just specifying their name and type and almost nothing else.

#### Data Container Classes

In this case, we wish to display the unconnected connectors.
Each connector belongs to a specific Revit element, and in some circumstances we wish to present the element data independently of its connectors, so I implemented two separate data containers for each, a base ElementData class and a derived ConnectorData one.
Both of them have a list of public properties whose main purpose is to define exactly what ultimately gets displayed in the data grid view.
They also each have a constructor which populates their public properties, and a ToString method to stream them to a text file for logging purposes:

```csharp
public class ElementData
{
  public string Class { get; set; }
  public string Category { get; set; }
  public string Family { get; set; }
  public string Symbol { get; set; }
  public string Name { get; set; }
  public int Id { get; set; }

  public ElementData(
    Element e,
    Document doc )
  {
    Class = e.GetType().Name;

    Category = ( null == e.Category )
      ? string.Empty
      : e.Category.Name;

    ElementId typeId = e.GetTypeId();

    ElementType elementType = ( null == typeId )
      ? null
      : doc.get\_Element( typeId ) as ElementType;

    FamilyInstance fi = e as FamilyInstance;
    Duct duct = e as Duct;
    if( null != duct )
    {
      string s = duct.DuctType.Name;
    }

    Family = ( null != fi )
      ? fi.Symbol.Family.Name
      : string.Empty;

    Symbol = ( null != fi ) ? fi.Symbol.Name
      : ( ( null != elementType ) ? elementType.Name
      : string.Empty );

    Name = e.Name;

    Id = e.Id.IntegerValue;
  }

  public override string ToString()
  {
    string c = (0 == Category.Length)
      ? Class
      : Category;

    string fam = (0 == Family.Length)
      ? string.Empty
      : "'" + Family + "' ";

    return string.Format(
      "{0} {1}<{2} '{3}'>",
      c, fam, Id, Name );
  }
}

public class ConnectorData : ElementData
{
  XYZ \_p;

  public string ConnectorType { get; set; }
  public string X { get; set; }
  public string Y { get; set; }
  public string Z { get; set; }

  public ConnectorData(
    Element e,
    Document doc,
    ConnectorType ct,
    XYZ p )
    : base( e, doc )
  {
    ConnectorType = ct.ToString();
    \_p = p;
    X = Util.RealString( p.X );
    Y = Util.RealString( p.Y );
    Z = Util.RealString( p.Z );
  }

  public override string ToString()
  {
    return string.Format(
      "{0} connector at ({1},{2},{3}) on {4}",
      ConnectorType, X, Y, Z, base.ToString() );
  }
}
```

There are a few details here worth mentioning.
For instance, I split the connector point into separate X, Y and Z coordinate values, because then I can later sort them based on these values, which gives me more sorting options and flexibility than if they were all lumped together into one single point data item.
Furthermore, I save the coordinates as string values instead of double numbers, because later on I do not want to see reams of useless, confusing and unintelligible post-comma digits in my user interface.

Talking about sorting, by the way:

#### Sortable Binding List

In the
[linked elements](http://thebuildingcoder.typepad.com/blog/2009/02/list-linked-elements.html) example
mentioned above, we simply used a generic .NET List<> as a data container.
That works fine, but does not automatically provide support for one additional neat feature available from the data grid view, which is the ability to sort the columns.

If we use a
[sortable binding list](http://www.timvw.be/presenting-the-sortablebindinglistt-take-two) instead,
all of our columns will automatically be sortable.
So we do just that.

#### Log File

Since most MEP models will initially contain a large number of parts, and many of them may have unconnected connectors, it makes sense to log the data to a text file in addition to presenting it in the modeless dialogue box.
The text file is easier than a form to manually search or automatically post-process for specific purposes.
Here is my log file implementation:

```python
class JtLogFile : IDisposable
{
  string \_path;
  StreamWriter \_sw;

  public JtLogFile( string basename )
  {
    \_path = System.IO.Path.Combine(
      System.IO.Path.GetTempPath(),
      basename + ".log" );

    \_sw = new StreamWriter( \_path, true );

    \_sw.WriteLine( "\n\rStart analysis {0}\n\r",
      DateTime.Now.ToString( "u" ) );
  }

  public void Dispose()
  {
    \_sw.Close();
    \_sw.Dispose();
  }

  public void Log( string s )
  {
    \_sw.WriteLine( s );
    Debug.WriteLine( s );
  }

  public string Path
  {
    get
    {
      return \_path;
    }
  }
}
```

#### Collecting and Displaying Information

We now have all the components ready to collect and display the unconnected connector information, both in the log file and the modeless form.
Enter the external command mainline Execute method, which performs the following steps:

- Determine Revit application window handle.- Retrieve all MEP elements that have connectors.- Loop over all elements and retrieve loose connectors.- List these in a log file and a data container.- Display the log file.- Start a modeless dialogue with entries for all loose connectors and set up the appropriate interaction with Revit.
            I'll postpone the discussion of this last step to a later post, so that we can give it the full attention that it deserves.

There is whole bunch of additional category tracking going on for logging and validation purposes.
At the end of the log file, we display information on the number of elements and connectors processed and what categories they belong to.
Don't let that confuse or distract you from the main points.

```python
public Result Execute(
  ExternalCommandData commandData,
  ref String message,
  ElementSet elements )
{
  // set up IWin32Window instance encapsulating
  // main Revit application window handle:

  if( null == \_hWndRevit )
  {
    Process process = Process.GetCurrentProcess();

    IntPtr h = process.MainWindowHandle;

    \_hWndRevit = new JtWindowHandle( h );
  }

  UIApplication app = commandData.Application;
  Document doc = app.ActiveUIDocument.Document;
  bool include\_wires = false;

  SortableBindingList<ConnectorData> data
    = new SortableBindingList<ConnectorData>();

  string path;

  using( JtLogFile log
    = new JtLogFile( "LooseConnectors" ) )
  {
    FilteredElementCollector collector
      = GetConnectorElements( doc, include\_wires );

    ConnectorSet connectors = null;

    Dictionary<string, List<Element>> categories
      = new Dictionary<string, List<Element>>();

    int nErrors = 0;
    int nUnconnected = 0;

    foreach( Element e in collector )
    {
      Category cat = e.Category;

      Debug.Assert( null != cat,
        "expected a valid category on all elements" );

      string key = cat.Name;

      if( !categories.ContainsKey( key ) )
      {
        categories[key] = new List<Element>();
      }
      categories[key].Add( e );

      connectors = null;

      try
      {
        connectors = GetConnectors( e );
      }
      catch( Exception ex )
      {
        ++nErrors;

        log.Log( string.Format(
          "Error {0} retrieving connectors "
          + "from {1}: {2} '{3}'",
          nErrors,
          new ElementData( e, doc ),
          ex.GetType().Name,
          ex.Message ) );
      }

      if( null != connectors )
      {
        foreach( Connector c in connectors )
        {
          if(

            // this is too restrictive:

            // ConnectorType.PhysicalConn
            //   == c.ConnectorType

            // i had to add some strange checks to avoid
            // IsConnected throwing an exception:

            ConnectorType.LogicalConn != c.ConnectorType
            && 32 != ((int)c.ConnectorType)
            && !c.IsConnected )
          {
            ++nUnconnected;

            ConnectorData cd = new ConnectorData(
              e, doc, c.ConnectorType, c.Origin );

            log.Log( string.Format(
              "Unconnected {0}: {1}",
              nUnconnected, cd ) );

            data.Add( cd );
          }
        }
      }
    }

    int total = categories.Values
        .Aggregate<List<Element>, int>(
          0, ( n, a ) => n + a.Count );

    log.Log( string.Format(
      "Examined {0} elements of {1} categories:",
      total, categories.Count ) );

    List<string> keys = new List<string>(
      categories.Keys );

    keys.Sort();

    foreach( string key in keys )
    {
      Element e = categories[key][0];

      BuiltInCategory bic = ( BuiltInCategory )
        e.Category.Id.IntegerValue;

      log.Log( string.Format(
        "{0,8} '{1}' {2}",
        categories[key].Count,
        key,
        bic ) );
    }

    log.Log( string.Format(
      "Error retrieving connectors on {0} elements,"
      + " {1} unconnected connectors found.",
      nErrors, nUnconnected ) );

    path = log.Path;
  }

  // display log file:

  Process.Start( path );

  // display data in modeless form and ensure
  // that the form remains on tp of Revit:

  LooseConnectorNavigator navigator
    = new LooseConnectorNavigator(
      data,
      new SetElementId( SetPendingElementId )  );

  navigator.Show( \_hWndRevit );

  // subscribe to Idling event:

  app.Idling += new EventHandler<IdlingEventArgs>(
    OnIdling );

  return Result.Succeeded;
}
```

Notes:

- I had to add some checks to work around an exception being thrown by the IsConnected predicate.
- I can use Process.Start to display the log file.

Here are a couple of sample lines from the generated log file listing the unconnected connector information in the rme\_basic\_sample\_project.rvt included in the Revit MEP distribution:

```csharp
Unconnected 48: EndConn connector at (77.22,-12.31,10.38)
on Ducts <411599 'Mitered Elbows / Taps'>
Unconnected 49: EndConn connector at (106.09,-12.31,10.38)
on Ducts <411618 'Mitered Elbows / Taps'>
Unconnected 50: EndConn connector at (136.28,-9.69,10.38)
on Ducts <411724 'Mitered Elbows / Taps'>
```

This is the summary information listed at the end of the log file:

```csharp
Examined 3927 elements of 15 categories:
309 'Air Terminals' OST\_DuctTerminal
14 'Conduit Fittings' OST\_ConduitFitting
20 'Conduits' OST\_Conduit
933 'Duct Fittings' OST\_DuctFitting
727 'Ducts' OST\_DuctCurves
29 'Electrical Equipment' OST\_ElectricalEquipment
424 'Electrical Fixtures' OST\_ElectricalFixtures
63 'Lighting Devices' OST\_LightingDevices
410 'Lighting Fixtures' OST\_LightingFixtures
47 'Mechanical Equipment' OST\_MechanicalEquipment
464 'Pipe Fittings' OST\_PipeFitting
467 'Pipes' OST\_PipeCurves
11 'Plumbing Fixtures' OST\_PlumbingFixtures
3 'Specialty Equipment' OST\_SpecialityEquipment
6 'Sprinklers' OST\_Sprinklers
Error retrieving connectors on 3 elements,
381 unconnected connectors found.
```

The modeless form populated and displayed by the command looks like this:

![Modeless loose connector form](img/modeless_loose_connector_form.png)

#### Next

The next and final instalment of this discussion will present the details of the interaction of the modeless dialogue box with Revit.
The modeless form does not have access to the Revit API, since it is
[not asynchronously accessible](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html).
Happily, we can use the
[Idling event](http://thebuildingcoder.typepad.com/blog/2010/04/idling-event.html)
to create a reliable and seamless solution for that.

In spite of not having completed the entire discussion yet, there is nothing to stop me from sharing the Visual Studio solution and source code with you already now, so here it is in
[loose\_connectors\_5.zip](zip/loose_connectors_5.zip).
This is actually almost the same sample that I already shared in the discussion of hooking up a modeless dialogue with the
[Revit parent window](zip/http://thebuildingcoder.typepad.com/blog/2010/06/revit-parent-window.html).

Enjoy, and I look forward to hearing back from you after my holidays!