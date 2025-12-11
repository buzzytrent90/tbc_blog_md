---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.2
content_type: code_example
optimization_date: '2025-12-11T11:44:14.257380'
original_url: https://thebuildingcoder.typepad.com/blog/0609_mep_placeholder.html
post_number: 0609
reading_time_minutes: 8
series: mep
slug: mep_placeholder
source_file: 0609_mep_placeholder.htm
tags:
- csharp
- elements
- family
- filtering
- levels
- python
- revit-api
- transactions
- views
- mep
title: MEP Placeholders
word_count: 1607
---

### MEP Placeholders

Happy
[Independence Day](http://en.wikipedia.org/wiki/Independence_Day_%28United_States%29) to
all US-Americans, and a wonderful Monday and beginning of the new week to all the rest of us!

![Independence Day](img/independence_day.jpg)

I recently discussed the
[Revit MEP 2012 API news](http://thebuildingcoder.typepad.com/blog/2011/06/the-revit-mep-2012-api.html) and
mentioned that MEP placeholders are one of the important new features in both the product and the API.

Mostly, new API features are demonstrated by some of the samples provided by the Revit SDK, but the placeholder elements form an exception to that rule.

Still, we did present an example of using them programmatically at
[DevDays 2010](http://thebuildingcoder.typepad.com/blog/2011/06/the-revit-mep-2012-api.html#2) and in the
[Revit 2012 API webcast](http://thebuildingcoder.typepad.com/blog/2011/06/the-revit-mep-2012-api.html#2).

Since this is a pretty neat little sample application and shows off some other nice API features besides the placeholder elements, I think it is worthwhile picking it up here in splendid isolation.
That also gives me a chance to polish the code once again, in preparation for one of my upcoming Autodesk University 2011 classes, CP4453 "Everything in Place with Revit MEP Programming", an update of the material I already presented at
[AU 2010](http://thebuildingcoder.typepad.com/blog/2010/11/autodesk-university-2010-class-materials.html) and
[2009](http://thebuildingcoder.typepad.com/blog/2009/10/au-2009.html).

So, to get back to the placeholder sample, it defines the following two commands:

1. [CreatePlaceholders](#1) – Demonstrate the NewMechanicalSystem and Duct.CreatePlaceholder methods in conjunction with NewElbowFitting and NewCrossFitting.- [ConvertPlaceholders](#2) – Convert placeholder elements to real ducts.

Besides the placeholder functionality, it also shows off some other nifty generic Revit API stuff, such as:

- Use of numerous generic collection methods and predicate functions to test for conditions in conjunction with Revit filtered element collectors.- Use of a transaction group with a number of transactions grouped in it.- The add-in manifest [limits the command visibility to Revit MEP](#3).

#### 1. Create Placeholder Elements

The first command, to create the placeholder elements, makes use of a helper class EquipmentElement to encapsulate the required equipment element data, including its supply air connector and its location and direction:
```csharp
class EquipmentElement
{
  public FamilyInstance FamilyInstance;
  public Connector SupplyAirConnector;
  public XYZ ConnectionPoint;
  public XYZ ConnectionDirection;

  public EquipmentElement(
    FamilyInstance familyInstance,
    Connector supplyAirConnector,
    XYZ connectionPoint,
    XYZ connectionDirection )
  {
    FamilyInstance = familyInstance;
    SupplyAirConnector = supplyAirConnector;
    ConnectionPoint = connectionPoint;
    ConnectionDirection = connectionDirection;
  }
}
```

Determination of the duct connectors and their matching neighbour connector is implemented by the GetDuctConnectorAt method, which returns the pair of connectors on a duct at a given location:
```csharp
static Connector GetDuctConnectorAt(
  Duct duct,
  XYZ location,
  out Connector otherConnector )
{
  otherConnector = null;

  Connector targetConnector = null;

  ConnectorManager cm = duct.ConnectorManager;

  foreach( Connector c in cm.Connectors )
  {
    if( c.Origin.IsAlmostEqualTo( location ) )
    {
      targetConnector = c;
    }
    else
    {
      otherConnector = c;
    }
  }
  return targetConnector;
}
```

Creating a new duct system with the selected equipment and placeholder elements for the ducts and fittings hooking them all together requires the following steps:

- Find duct type to use.- Find Level 1 to place ductwork.- Find all mechanical equipment elements.- Determine their connectors and set up EquipmentElement instances for them.- Determine a waypoint for the cross fitting from the XY centre point of all elements bounding boxes and a given height eleven feet over the floor.- Create a new MEP mechanical system element from all connectors.- Iterate over the equiment elements.- Create and connect placeholder ducts between them.- Add elbow and cross fittings.

Here is the source code for the mainline of the first command to create the placeholder elements implementing those steps, placing each step into its own transaction and assimilating all the transactions together into one single transaction group:
```python
static void ExecuteMepPlaceholders( Document doc )
{
  // Find duct type to use

  Func<Element, bool> isRectangularRadiusDuctType
    = dt => dt.Name.Contains( "Radius Elbows / Tees" );

  Element ductType
    = new FilteredElementCollector( doc )
      .OfClass( typeof( DuctType ) )
      .First<Element>( isRectangularRadiusDuctType );

  // Find Level 1 to place ductwork

  Func<Level, bool> isLevel1
    = level => level.Name.Equals( "Level 1" );

  Level level1 = new FilteredElementCollector( doc )
    .OfClass( typeof( Level ) )
    .Cast<Level>()
    .First<Level>( isLevel1 );

  // Find all mechanical equipment elements

  List<BuiltInCategory> cats
    = new List<BuiltInCategory>( 2 );

  cats.Add( BuiltInCategory.OST\_MechanicalEquipment );
  cats.Add( BuiltInCategory.OST\_DuctTerminal );

  FilteredElementCollector equipment
    = new FilteredElementCollector( doc )
      .OfClass( typeof( FamilyInstance ) )
      .WherePasses( new ElementMulticategoryFilter(
        cats ) );

  List<EquipmentElement> equipmentElements
    = new List<EquipmentElement>();

  foreach( FamilyInstance fi in equipment )
  {
    // find connectors

    ConnectorManager cm
      = fi.MEPModel.ConnectorManager;

    ConnectorSet cs = cm.Connectors;

    foreach( Connector c in cs )
    {
      if( Domain.DomainHvac == c.Domain
        && DuctSystemType.SupplyAir == c.DuctSystemType )
      {
        // connector point and direction

        XYZ p = c.Origin;
        XYZ v = c.CoordinateSystem.BasisZ;

        equipmentElements.Add(
          new EquipmentElement( fi, c, p, v ) );
      }
    }
  }

  // Determine a waypoint for the cross fitting
  // from the XY centre point of all elements
  // bounding boxes and a given height eleven
  // feet over the floor:

  double[] min = new double[] {
    double.PositiveInfinity,
    double.PositiveInfinity };

  double[] max = new double[] {
    double.NegativeInfinity,
    double.NegativeInfinity };

  foreach( EquipmentElement e in equipmentElements )
  {
    BoundingBoxXYZ box
      = e.FamilyInstance.get\_BoundingBox( null );

    for( int i = 0; i < 2; ++i )
    {
      min[i] = box.Min[i] < min[i] ? box.Min[i] : min[i];
      max[i] = box.Max[i] > max[i] ? box.Max[i] : max[i];
    }
  }

  double verticalLocation = level1.Elevation + 11.0;

  XYZ wayPoint = new XYZ(
    ( min[0] + max[0] ) / 2,
    ( min[1] + max[1] ) / 2,
    verticalLocation );

  Debug.Print( "Waypoint found at {0}",
    PointString( wayPoint ) );

  TransactionGroup tGroup
    = new TransactionGroup( doc );

  tGroup.Start( "Auto-route placeholders" );

  // Create a new MEP mechanical system
  // element from all connectors:

  Transaction t = new Transaction( doc );

  t.Start( "Create system" );

  Connector baseConnector = null;
  ConnectorSet newSystemCS = new ConnectorSet();

  foreach( EquipmentElement e in equipmentElements )
  {
    if( e.FamilyInstance.Category.Id.Equals(
      new ElementId( BuiltInCategory.OST\_MechanicalEquipment ) ) )
    {
      baseConnector = e.SupplyAirConnector;
    }
    else
    {
      newSystemCS.Insert( e.SupplyAirConnector );
    }
  }
  doc.Create.NewMechanicalSystem( baseConnector,
    newSystemCS, DuctSystemType.SupplyAir );

  t.Commit();

  bool xFirst = true;

  List<Connector> wayPointConnectors
    = new List<Connector>();

  foreach( EquipmentElement e in equipmentElements )
  {
    Connector nextConnector;

    // if connector direction is vertical,
    // add duct to reach target elevation

    if( !e.ConnectionDirection.IsAlmostEqualTo( XYZ.BasisZ ) )
    {
      throw new NotImplementedException(
        "Not implemented for initially non-vertical connectors" );
    }

    t.Start( "Create placeholder duct" );

    XYZ secondPoint = new XYZ( e.ConnectionPoint.X,
      e.ConnectionPoint.Y, wayPoint.Z );

    Duct duct = Duct.CreatePlaceholder( doc,
      ductType.Id, level1.Id, e.ConnectionPoint,
      secondPoint );

    t.Commit();

    t.Start( "Connect duct" );

    Connector targetConnector = GetDuctConnectorAt(
      duct, e.ConnectionPoint, out nextConnector );

    targetConnector.ConnectTo( e.SupplyAirConnector );

    t.Commit();

    // all connections should make a right
    // hand turn into the waypoint

    XYZ nextConnectorPoint = nextConnector.Origin;
    XYZ nextWayPoint = null;
    if( xFirst )
    {
      nextWayPoint = new XYZ( wayPoint.X,
        nextConnectorPoint.Y, wayPoint.Z );
    }
    else
    {
      nextWayPoint = new XYZ( nextConnectorPoint.X,
        wayPoint.Y, wayPoint.Z );
    }

    t.Start( "Create placeholder duct" );

    Duct nextDuct = Duct.CreatePlaceholder( doc,
      ductType.Id, level1.Id, nextConnectorPoint,
      nextWayPoint );

    t.Commit();

    t.Start( "Add fitting" );

    Connector nextNextConnector;

    Connector nextTargetConnector
      = GetDuctConnectorAt( nextDuct,
        nextConnectorPoint, out nextNextConnector );

    doc.Create.NewElbowFitting( nextConnector,
      nextTargetConnector );

    t.Commit();

    t.Start( "Create placeholder duct" );

    nextDuct = Duct.CreatePlaceholder( doc,
      ductType.Id, level1.Id,
      nextNextConnector.Origin, wayPoint );

    t.Commit();

    t.Start( "Add fitting" );

    Connector lastConnector;

    Connector nextNextTargetConnector
      = GetDuctConnectorAt( nextDuct,
        nextNextConnector.Origin,
        out lastConnector );

    doc.Create.NewElbowFitting( nextNextConnector,
      nextNextTargetConnector );

    wayPointConnectors.Add( lastConnector );

    t.Commit();

    xFirst = !xFirst;
  }

  if( wayPointConnectors.Count != 4 )
  {
    throw new Exception(
      "Unexpected number of connectors" );
  }

  t.Start( "Add cross fitting" );

  doc.Create.NewCrossFitting(
    wayPointConnectors[0], wayPointConnectors[2],
    wayPointConnectors[1], wayPointConnectors[3] );

  t.Commit();

  tGroup.Assimilate();
}
```

#### 2. Convert Placeholder Elements

The conversion of placeholder elements to real ductwork is much simpler, of course.
The following code collects all placeholder elements, and calls ConvertDuctPlaceholders to convert them.
Again, we make use of a generic predicate function to find the placeholder elements, and generic collection methods to create a list of their element ids:
```csharp
public Result Execute(
  ExternalCommandData commandData,
  ref string message,
  ElementSet elements )
{
  Document doc = commandData.View.Document;

  Transaction t = new Transaction( doc );
  t.Start( "Convert placeholder network" );

  FilteredElementCollector ductCollector
    = new FilteredElementCollector( doc )
      .OfClass( typeof( Duct ) );

  Func<Duct, bool> isPlaceholder
    = duct => duct.IsPlaceholder;

  IEnumerable<Duct> ducts = ductCollector
    .OfType<Duct>()
    .Where<Duct>( isPlaceholder );

  ICollection<ElementId> ductIds = ducts
    .Select<Duct, ElementId>( duct => duct.Id )
    .ToList<ElementId>();

  MechanicalUtils.ConvertDuctPlaceholders(
    doc, ductIds );

  t.Commit();

  return Result.Succeeded;
}
```

#### 3. Command Visibility in MEP only

As mentioned, the two commands are only visible in Revit MEP, because the add-in manifest specifies the following VisibilityMode tags:
```csharp
<?xml version="1.0" encoding="utf-8"?>
<RevitAddIns>
  <AddIn Type="Command">
    <Text>Create MEP Placeholders</Text>
    <Description>Create MEP placeholder elements</Description>
    <Assembly>C:\a\lib\revit\2012\adn\webcast\src\MepPlaceholders\MepPlaceholders\bin\Debug\MepPlaceholders.dll</Assembly>
    <FullClassName>MepPlaceholders.CreatePlaceholders</FullClassName>
    <ClientId>54b7cf02-64bd-4af5-a701-030a81e6c0d5</ClientId>
    <VendorId>ADNP</VendorId>
    <VendorDescription>Autodesk, Inc. www.autodesk.com</VendorDescription>
    <VisibilityMode>NotVisibleInArchitecture</VisibilityMode>
    <VisibilityMode>NotVisibleInFamily</VisibilityMode>
    <VisibilityMode>NotVisibleInStructure</VisibilityMode>
    <VisibilityMode>NotVisibleWhenNoActiveDocument</VisibilityMode>  </AddIn>
  <AddIn Type="Command">
    <Text>Convert MEP Placeholders</Text>
    <Description>Convert MEP placeholder elements</Description>
    <Assembly>C:\a\lib\revit\2012\adn\webcast\src\MepPlaceholders\MepPlaceholders\bin\Debug\MepPlaceholders.dll</Assembly>
    <FullClassName>MepPlaceholders.ConvertPlaceholders</FullClassName>
    <ClientId>2c834e62-3d55-4aae-95d9-5646a74dmaroon6</ClientId>
    <VendorId>ADNP</VendorId>
    <VendorDescription>Autodesk, Inc. www.autodesk.com</VendorDescription>
    <VisibilityMode>NotVisibleInArchitecture</VisibilityMode>
    <VisibilityMode>NotVisibleInFamily</VisibilityMode>
    <VisibilityMode>NotVisibleInStructure</VisibilityMode>
    <VisibilityMode>NotVisibleWhenNoActiveDocument</VisibilityMode>
  </AddIn>
</RevitAddIns>
```

#### Sample Project

Here is a sample project providing a starting point for the command to run in:

![MEP placeholder initial starting point](img/mep_placeholder01.png)

After creation of the placeholder elements, all location points have been calculated and a complete well-connected system defined:

![MEP placeholder elements form a complete system](img/mep_placeholder02.png)

As soon as the exact element types to use are known, the placeholders can be converted to real ductwork:

![MEP placeholder elements converted to ductwork](img/mep_placeholder03.png)

Here is [MepPlaceholders.zip](zip/MepPlaceholders.zip) containing the complete MEP placeholder sample source code, Visual Studio solution and add-in manifest.