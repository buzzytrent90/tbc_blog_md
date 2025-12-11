---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.0
content_type: qa
optimization_date: '2025-12-11T11:44:16.775557'
original_url: https://thebuildingcoder.typepad.com/blog/1802_electrical_load.html
post_number: '1802'
reading_time_minutes: 3
series: mep
slug: electrical_load
source_file: 1802_electrical_load.md
tags:
- elements
- family
- filtering
- parameters
- references
- revit-api
- selection
- sheets
- transactions
- views
- mep
title: Electrical Load
word_count: 598
---

### Retrieving the Electrical Load from a Fixture
Today is my birthday, so I am trying hard to work less.
I am not completely successful, though, I'm afraid.
Very kindly, Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) [@CADBIMDeveloper](https://github.com/CADBIMDeveloper) Ignatovich, aka Александр Игнатович,
provided us all with a gift for the day in the form of a new external
command [CmdElectricalLoad](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdElectricalLoad.cs) that he submitted
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
in [pull request #12](https://github.com/jeremytammik/the_building_coder_samples/pull/12), saying:
> I recently answered a question on
the [Russian ADN forum](https://adn-cis.org/forum/index.php) on how to retrieve the value from the `Load` column in the electrical system browser for a specific fixture family instance,
[активная электрическая мощность элемента](https://adn-cis.org/forum/index.php?topic=9577) (in English, [active electric power of an element](https://adn-cis.org/forum/index.php?topic=9577)):
![Electrical load](img/electrical_load.png)
\*\*Question:\*\* Tell me how to get the built-in parameter of the active power of an element in the category of electrical appliances `OST_ElectricalFixtures`.
RevitLookup provides information only on electrical circuits.
I am searching for the value of the power parameter, but it is not visible in the element.
If you go into `ElectricalSystem`, it starts working with circuits, and I need the power of the elements.
\*\*Answer:\*\* First, you need to get the necessary connector from the family related to the system of interest:
```csharp
var mepConnectorInfo = connector.GetMEPConnectorInfo();
```
From that, we can get the value of the `ParameterValue` parameter:
```csharp
var parameterValue
= ( DoubleParameterValue ) mepConnectorInfo
.GetConnectorParameterValue ( new ElementId (
BuiltInParameter . RBS_ELEC_APPARENT_LOAD ) );
```
And the final step:
```csharp
var value = UnitUtils.ConvertFromInternalUnits(
parameterValue.Value, DisplayUnitType.DUT_WATTS );
```
Thank you very much, Alexander, for answering this important question and sharing the solution in such a useful form!
I edited Alexander's new command and added it to the text file `BcSamples.txt` for loading it using the Revit SDK RvtSamples application∫®.
Here is the resulting code, including Alexander's inimitable use of advanced .NET functionality enabling extremely succinct code:
#### CmdElectricalLoad
```csharp
[Transaction( TransactionMode.Manual )]
public class CmdElectricalLoad : IExternalCommand
{
class ElectricalApparentLoad
{
public ElectricalApparentLoad(
ElectricalSystemType electricalSystemType,
int connectorId,
double apparentLoad )
{
ElectricalSystemType = electricalSystemType;
ConnectorId = connectorId;
ApparentLoad = apparentLoad;
}
public ElectricalSystemType ElectricalSystemType { get; }
public int ConnectorId { get; }
public double ApparentLoad { get; }
public override string ToString() =>
$"{ElectricalSystemType}: {ConnectorId} "
+ "- {ApparentLoad} V\*A";
}
class ElectricalApparentLoadFactory
{
public IEnumerable
Create( FamilyInstance familyInstance )
{
return familyInstance.MEPModel
.ConnectorManager
.Connectors
.Cast()
.Select( Create )
.Where( x => x != null );
}
private static ElectricalApparentLoad Create(
Connector connector )
{
if( connector.Domain != Domain.DomainElectrical )
return null;
var mepConnectorInfo
= connector.GetMEPConnectorInfo()
as MEPFamilyConnectorInfo;
var parameterValue = mepConnectorInfo
?.GetConnectorParameterValue(
new ElementId(
BuiltInParameter.RBS_ELEC_APPARENT_LOAD ) )
as DoubleParameterValue;
if( parameterValue == null )
return null;
var load = UnitUtils.ConvertFromInternalUnits(
parameterValue.Value,
DisplayUnitType.DUT_VOLT_AMPERES );
return new ElectricalApparentLoad(
connector.ElectricalSystemType,
connector.Id, load );
}
}
class FamilyInstanceWithApparentLoadSelectionFilter
: ISelectionFilter
{
private readonly ElectricalApparentLoadFactory
electricalApparentLoadFactory;
public FamilyInstanceWithApparentLoadSelectionFilter(
ElectricalApparentLoadFactory
electricalApparentLoadFactory )
{
this.electricalApparentLoadFactory
= electricalApparentLoadFactory;
}
public bool AllowElement( Element elem )
{
var familyInstance = elem as FamilyInstance;
if( familyInstance == null )
return false;
return electricalApparentLoadFactory
.Create( familyInstance )
.Any();
}
public bool AllowReference( Reference r, XYZ p )
=> false;
}
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
var app = commandData.Application;
var uidoc = app.ActiveUIDocument;
var familyInstance
= SelectFamilyInstanceWithApparentLoad(
uidoc );
if( familyInstance == null )
return Result.Cancelled;
var electricalApparentLoadFactory
= new ElectricalApparentLoadFactory();
var apparentLoads = electricalApparentLoadFactory
.Create( familyInstance );
TaskDialog.Show( "CmdElectricalLoad",
string.Join( "\n", apparentLoads ) );
return Result.Succeeded;
}
private static FamilyInstance
SelectFamilyInstanceWithApparentLoad(
UIDocument uidoc )
{
var electricalApparentLoadFactory
= new ElectricalApparentLoadFactory();
var selectionFilter
= new FamilyInstanceWithApparentLoadSelectionFilter(
electricalApparentLoadFactory );
try
{
return (FamilyInstance) uidoc.Document.GetElement(
uidoc.Selection.PickObject( ObjectType.Element,
selectionFilter ) );
}
catch( OperationCanceledException )
{
return null;
}
}
}
```