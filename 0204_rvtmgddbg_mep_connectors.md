---
post_number: "0204"
title: "Fixing RvtMgdDbg for MEP Connectors"
slug: "rvtmgddbg_mep_connectors"
author: "Jeremy Tammik"
tags: ['csharp', 'revit-api', 'windows']
source_file: "0204_rvtmgddbg_mep_connectors.htm"
original_url: "https://thebuildingcoder.typepad.com/blog/0204_rvtmgddbg_mep_connectors.html"
---

### Fixing RvtMgdDbg for MEP Connectors

Still working on preparations for the MEP webcast mentioned
[last week](http://thebuildingcoder.typepad.com/blog/2009/08/mep-sample-ribbon-panel.html),
I notice that RvtMgdDbg is having trouble displaying the connectors on electrical components, throwing an exception saying

```
Angle is available only for connectors of DomainHavc and DomainPiping.
```

The method I describe here to fix this problem can be used by anyone who runs into a problem with RvtMgdDbg.
It is not limited to the case of MEP connectors, and intended more as an example and a how-to on how to fix RvtMgdDbg to display specific information that you are interested in.

Luckily, RvtMgdDbg is provided in source code format, and everybody can debug it and fix problems for themselves as they see fit.
In my case, I debugged the RvtMgdDbg code and traced the problem to the implementation of the overloaded method Stream( ArrayList data, Connector connector ) in the module
Snoop > CollectorExts > CollectorExtMEP.cs.
The fix is to add an exception handler to the offending property.
In this case, the original code simply went ahead and blindly accessed the Angle property:

```csharp
data.Add( new Snoop.Data.Double( "Angle", connector.Angle ) );
```

As the exception message says, this access is prohibited unless the connector is defined for the HVAC or piping domain.
I therefore added a check to test the connector domain before accessing it, and the updated code now reads:
```csharp
if( Domain.DomainHvac == connector.Domain
  || Domain.DomainPiping == connector.Domain )
{
  data.Add( new Snoop.Data.Double( "Angle",
    connector.Angle ) );
}
```

That solves that little problem.

Another way to handle this would be to simply encapsulate the offending line of code into its own individual little exception handler, but that would add considerable unnecessary execution overhead.
Remember, an exception should be used only for unexpected, exceptional circumstances.
If a property is expected to fail under certain circumstances, then these should be caught using traditional and more effective means.

Unfortunately, there are quite a few more properties on the connector class which are similarly limited and throw exceptions if called under invalid circumstances, so the method Stream(ArrayList data, Connector connector) could use quite a few additional corrections.
For instance, the Radius property may only be queried in the case of a connector with a round cross section.

Since there are so many properties and I do not want to add individual checks for each of them, I have actually resorted to adding the individual exception handlers I mentioned above to each property for the time being, simply in order to use RvtMgdDbg for electrical connectors right away with no further ado.

I encapsulated each line of code accessing a property into an exception handler using the Visual Studio regular expression replacement feature, searching for a regular expression like this:

```
{data\.Add.*, connector\.}{[^\.\)]*}{[\)\.].*}\n
```

The replacement code looks like this:

```
try\n{\n\1\2\3\n}\ncatch\n{\nDebug.Print( "\2: "+ex.Message );\n}\n
```

Visual Studio automatically adds the proper indenting for me if I cut out the generated code and then paste it right back into the same place again.

For example, here is the unprotected call to access the AssignedDuctFlowConfiguration property:
```csharp
  data.Add( new Snoop.Data.String(
    "Duct flow configuration type",
    connector.AssignedDuctFlowConfiguration.ToString() ) );
```

The result of executing the regular expression search and replace operation on that single line of code is to convert the unprotected property access to one encapsulated within its own exception handler.
Here is the resulting protected AssignedDuctFlowConfiguration property access:

```csharp
  try
  {
    data.Add( new Snoop.Data.String(
      "Duct flow configuration type",
      connector.AssignedDuctFlowConfiguration.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedDuctFlowConfiguration: "
      + ex.Message );
  }
```

Here is the complete code of my modified version of the overloaded method Stream( ArrayList data, Connector connector ):

```csharp
private void
Stream(ArrayList data, Connector connector)
{
  data.Add(new Snoop.Data.ClassSeparator(typeof(Connector)));

  try
  {
    data.Add( new Snoop.Data.Enumerable( "All refs", connector.AllRefs ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AllRefs: " + ex.Message );
  }
  if( Domain.DomainHvac == connector.Domain
    || Domain.DomainPiping == connector.Domain )
  {
    data.Add( new Snoop.Data.Double( "Angle", connector.Angle ) );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Duct flow configuration type", connector.AssignedDuctFlowConfiguration.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedDuctFlowConfiguration: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Duct loss method type", connector.AssignedDuctLossMethod.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedDuctLossMethod: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Assigned fixture units", connector.AssignedFixtureUnits ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedFixtureUnits: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Assigned flow", connector.AssignedFlow ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedFlow: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Assigned flow direction", connector.AssignedFlowDirection.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedFlowDirection: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Assigned flow factor", connector.AssignedFlowFactor ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedFlowFactor: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Assigned K coefficient", connector.AssignedKCoefficient ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedKCoefficient: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Assigned loss coefficient", connector.AssignedLossCoefficient ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedLossCoefficient: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Assigned pipe flow configuration", connector.AssignedPipeFlowConfiguration.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedPipeFlowConfiguration: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Assigned pipe loss method", connector.AssignedPipeLossMethod.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedPipeLossMethod: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Assigned pressure drop", connector.AssignedPressureDrop ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "AssignedPressureDrop: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Coefficient", connector.Coefficient ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Coefficient: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Object( "Connector manager", connector.ConnectorManager ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "ConnectorManager: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Connector type", connector.ConnectorType.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "ConnectorType: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Object( "Coordinate system", connector.CoordinateSystem ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "CoordinateSystem: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Demand", connector.Demand ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Demand: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Direction", connector.Direction.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Direction: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Domain", connector.Domain.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Domain: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Duct system type", connector.DuctSystemType.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "DuctSystemType: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Electrical system type", connector.ElectricalSystemType.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "ElectricalSystemType: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Flow", connector.Flow ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Flow: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Height", connector.Height ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Height: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Bool( "Is connected", connector.IsConnected ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "IsConnected: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Bool( "Is movable", connector.IsMovable ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "IsMovable: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Object( "MEP system", connector.MEPSystem ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "MEPSystem: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Xyz( "Origin", connector.Origin ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Origin: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Pipe system type", connector.PipeSystemType.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "PipeSystemType: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Pressure drop", connector.PressureDrop ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "PressureDrop: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Velocity pressure", connector.VelocityPressure ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "VelocityPressure: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Width", connector.Width ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Width: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Double( "Radius", connector.Radius ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Radius: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.String( "Shape", connector.Shape.ToString() ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Shape: " + ex.Message );
  }
  try
  {
    data.Add( new Snoop.Data.Object( "Owner", connector.Owner ) );
  }
  catch( System.NotSupportedException ex )
  {
    Debug.Print( "Owner: " + ex.Message );
  }
}
```

If you wish to examine the truncated overly long lines, you can copy and paste them to an editor.

Running the modified version of RvtMgdDbg and selecting an electrical connector produces the following list in the Visual Studio debug output window of all the properties that are throwing exceptions:

```
AssignedDuctFlowConfiguration: Assigned duct flow configuration is available only for connectors of DomainPiping.
AssignedDuctLossMethod: Assigned duct loss method is available only in connectors of DomainHvac.
AssignedFixtureUnits: Assigned fixture units available only for connectors of DomainPiping.
AssignedFlow: Assigned flow is available only for connectors of DomainHavc or DomainPiping.
AssignedFlowDirection: Assigned flow direction is available only for connectors of DomainHavc and DomainPiping.
AssignedFlowFactor: Assigned flow factor is available only for connectors of DomainPiping or DomainHvac.
AssignedKCoefficient: Assigned KCoefficient is available only for connectors of DomainPiping.
AssignedLossCoefficient: Assigned loss coefficient is available only for connectors of DomainHavc.
AssignedPipeFlowConfiguration: Assigned pipe flow configuration is available only for connectors of DomainPiping.
AssignedPipeLossMethod: Assigned pipe loss method is available only for connectors of DomainPiping.
AssignedPressureDrop: Assigned pressure drop is available only for connectors of DomainHavc and DomainPiping.
Coefficient: Coefficient is available only for connectors of DomainHavc and DomainPiping.
CoordinateSystem: Coordinate System is available only for connectors of PhysicalConn.
Demand: Demand is available only for connector of DomainPiping.
Direction: Direction is available only for connectors of DomainHvac and DoaminPiping.
DuctSystemType: Duct system type is available only for connectors of DomainHvac.
ElectricalSystemType: Electrical system type is available only for connectors of DomainElectrical.
Flow: Flow is available only for connectors of DomainHavc and DomainPiping.
Height: The shape of the connector is not rectangular.
IsConnected: Connection status is available only for connectors of PhysicalConn type.
Origin: Origin is available only for connectors of PhysicalConn type.
PipeSystemType: Pipe system type is available only for connectors of DomainPiping.
PressureDrop: Pressure drop is available only for connectors of DomainHavc and DomainPiping.
VelocityPressure: Velocity pressure is available only for connectors of DomainHavc and DomainPiping.
Width: The shape of the connector is not rectangular.
Radius: The shape of the connector is not round.
```

Again, copy and paste to an editor to see the complete truncated lines.

Since each exception caught by an invalid property access is caught individually and processing can continue with an attempt at the next property, the other desired result is that the property window is now successfully populated and displayed with the remaining properties that have not thrown an exception:

![RvtMgdDbg connector properties](img/rvtmgddbg_connector_1.jpg)

As pointed out above, this is only a partial and temporary fix.
In the long run, the individual exception handlers should be replaced by if clauses, as done for the Angle property.