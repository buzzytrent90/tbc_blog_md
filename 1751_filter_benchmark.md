---
post_number: "1751"
title: "Filter Benchmark"
slug: "filter_benchmark"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'levels', 'parameters', 'references', 'revit-api', 'selection', 'sheets', 'transactions', 'views']
source_file: "1751_filter_benchmark.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1751_filter_benchmark.html"
---

### Filtered Element Collector Benchmark
Today, let's present a benchmark monitoring filtered element collector performance.
First, however, a quick note on a very useful Forge learning resource:
- [Forge learning resource](#2)
- [Filtered element collector benchmark](#3)
![Stopwatch](img/stopwatch.png)
#### Forge Learning Resource
If you are new to Forge or want to dive in deeper, you can find a collection of very cool Forge training material
at [learnforge.autodesk.io](https://learnforge.autodesk.io), focusing specifically on BIM360 and design automation.
Here is the table of contents:
- Before you start coding
- Tools
- OAuth
- View your models
- Create a server
- Authenticate
- Upload file to OSS
- Translate the file
- Show on Viewer
- View BIM 360 & Fusion models
- Create a server
- Authorize
- List hubs & projects
- User information
- Show on Viewer
- Modify your models
- Create a server
- Basic app UI
- Prepare a plugin
- Define an activity
#### Filtered Element Collector Benchmark
Back to the Revit API, I recently reiterated the differences between [slow, slower still and faster filtering](https://thebuildingcoder.typepad.com/blog/2019/04/slow-slower-still-and-faster-filtering.html).
In the end, the only way to tell whether your filter is performing well or not is to implement
some [benchmarking](https://en.wikipedia.org/wiki/Benchmark_(computing)) for it.
Jai Hari Hara Sudhan very commendably did so, documenting his progress and sharing his results in
a [series of](https://thebuildingcoder.typepad.com/blog/2019/04/slow-slower-still-and-faster-filtering.html#comment-4421084673)
[comments on](https://thebuildingcoder.typepad.com/blog/2019/04/slow-slower-still-and-faster-filtering.html#comment-4421183783)
[that post](https://thebuildingcoder.typepad.com/blog/2019/04/slow-slower-still-and-faster-filtering.html#comment-4421443231) and in
his [API speed test screencast](https://knowledge.autodesk.com/community/screencast/99858ef7-c4c8-4599-ba6d-0394ff830d62) demonstrating
the benchmark running live.
Here is a summary of our discussion and his final benchmarking code:
\*\*Question:\*\* The above content is very useful.
I am using three methods to filter and select the element:
1. Using `FilterRule` method (filters in floor by floor / level)
2. Using `Factory` Method (filters in the projects)
3. Selection with interface (element by element)
Question 1. Which one is the best in performance (quick)? `FilterRule` or `Factory` method?
Question 2. In the FamilySelectionFilter method, is there any better or more performant method to select the elements?
\*\*Answer:\*\* Nobody can tell you beforehand how these different approaches will perform in your specific context.
I therefore suggest that you benchmark them yourself and let us know the result.
[The Building Coder topic groups](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5) lists
several benchmarking examples that you can look at to see how.
\*\*Response:\*\* I implemented a benchmark.
It is impossible to determine exact constant performance (time), because the results differ from run to run.
Please refer to my [API speed test screencast](https://knowledge.autodesk.com/community/screencast/99858ef7-c4c8-4599-ba6d-0394ff830d62).
Finally, I rank the different approaches as follows:

1. `Factory` method (average time = 766.1 microseconds)
   – 100%
2. `FilterRule` method (average time = 889.0 microseconds)
   – 116% slower than `Factory`
3. `Linq2` method (average time = 983.5 microseconds)
   – 128% slower than `Factory`
4. `Linq1` method (average time = 1173.3 microseconds)
   – 153% slower than `Factory`

The code as follows:
```csharp
[Transaction( TransactionMode.Manual )]
public class Elec_test : IExternalCommand
{
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiapp = commandData.Application;
UIDocument uidoc = uiapp.ActiveUIDocument;
Application app = uiapp.Application;
Document doc = uidoc.Document;
Selection sel = uidoc.Selection;
TaskDialog.Show( "BuilDTecH Architects",
"BuilDTecH Architects by Sudhan" );
InputData InputData = new InputData();
InputData.ShowDialog();
IList data = InputData.Data;
/////////////////////////////// Input Values
double LeftOffset = Convert.ToDouble( data[1] ),
RightOffset = Convert.ToDouble( data[2] ),
TopOffset = Convert.ToDouble( data[3] ),
BottomOffset = Convert.ToDouble( data[4] ),
NearClipOffset = -Convert.ToDouble( data[5] ),
FarClipOffset = Convert.ToDouble( data[6] );
string Section_Name = "Electrical GangBox",
ElectricalEquipment = "Modular Gang Box";
//Input /////////////////////////////// Input Values
if( data[0].Equals( InputData.Option.ByFloor ) )
{
Timer floortimeLinq1 = new Timer();
floortimeLinq1.Start();
IEnumerable elems = Linq1( doc,
BuiltInCategory.OST_ElectricalEquipment,
ElectricalEquipment );
floortimeLinq1.Stop();
TaskDialog.Show( "time", "LINQ1 Method Time = "
+ floortimeLinq1.Duration.ToString()
+ " No. of Elements = " + elems.Count().ToString() );
elems = null;
Timer floortimeLinq2 = new Timer();
floortimeLinq2.Start();
elems = Linq2( doc,
BuiltInCategory.OST_ElectricalEquipment,
ElectricalEquipment );
floortimeLinq2.Stop();
TaskDialog.Show( "time", "LINQ2 Method Time = "
+ floortimeLinq2.Duration.ToString()
+ " No. of Elements = " + elems.Count().ToString() );
elems = null;
Timer floortimeFilterRule = new Timer();
floortimeFilterRule.Start();
elems = FilterRule( doc, // uidoc.ActiveView.Id,
BuiltInCategory.OST_ElectricalEquipment,
ElectricalEquipment );
floortimeFilterRule.Stop();
TaskDialog.Show( "time", "Filter Rule Method Time = "
+ floortimeFilterRule.Duration.ToString()
+ " No. of Elements = " + elems.Count().ToString() );
elems = null;
Timer floortimeFactoryRule = new Timer();
floortimeFactoryRule.Start();
elems = Factory( doc,
BuiltInCategory.OST_ElectricalEquipment,
ElectricalEquipment );
floortimeFactoryRule.Stop();
TaskDialog.Show( "time", " Factory Rule Method Time = "
+ floortimeFactoryRule.Duration.ToString()
+ " No. of Elements = " + elems.Count().ToString() );
}
else if( data[0].Equals( InputData.Option.BySingle ) )
{
TaskDialog td = new TaskDialog( "Element By Element" );
td.Title = "Want to Continue";
td.MainInstruction = "Do you want to create a new section";
td.CommonButtons = TaskDialogCommonButtons.Yes
| TaskDialogCommonButtons.No;
td.DefaultButton = TaskDialogResult.Yes;
bool next = true;
while( next )
{
ISelectionFilter selFilter
= new FamilySelectionFilter( doc,
BuiltInCategory.OST_ElectricalEquipment,
ElectricalEquipment );
Reference refe = sel.PickObject(
ObjectType.Element, selFilter, "Select Object" );
Element ele = doc.GetElement( refe );
Draw_Section.Draw( doc, ele, Section_Name,
LeftOffset, RightOffset, TopOffset, BottomOffset,
NearClipOffset, FarClipOffset );
TaskDialogResult tdRes = td.Show();
if( tdRes == TaskDialogResult.No )
{ next = false; }
}
}
else if( data[0].Equals( InputData.Option.ByProject ) )
{
Timer floortimeFactoryRule = new Timer();
floortimeFactoryRule.Start();
IEnumerable elems = Factory( doc,
BuiltInCategory.OST_ElectricalEquipment,
ElectricalEquipment );
floortimeFactoryRule.Stop();
TaskDialog.Show( "time", " Factory Rule Method Time = "
+ floortimeFactoryRule.Duration.ToString()
+ " No. of Elements = " + elems.Count().ToString() );
foreach( Element ele in elems )
{
Draw_Section.Draw( doc, ele, Section_Name,
LeftOffset, RightOffset, TopOffset, BottomOffset,
NearClipOffset, FarClipOffset );
}
}
return Result.Succeeded;
}
public class FamilySelectionFilter : ISelectionFilter
{
Document Doc;
string FmlyName = "";
int BultCatId;
public FamilySelectionFilter(
Document doc,
BuiltInCategory BuiltInCat,
string familyTypeName )
{
Doc = doc;
FmlyName = familyTypeName;
BultCatId = (int) BuiltInCat;
}
public bool AllowElement( Element elem )
{
return elem.Category.Id.IntegerValue == BultCatId;
}
public bool AllowReference( Reference refer, XYZ point )
{
Element e = Doc.GetElement( refer );
return e.get_Parameter( BuiltInParameter.ELEM_FAMILY_PARAM )
.AsValueString().Equals( FmlyName );
}
}
#region Retrieve named family type using either LINQ or a parameter filter
static IEnumerable Linq1(
Document doc,
BuiltInCategory BultCat,
string familyTypeName )
{
return new FilteredElementCollector( doc ).OfCategory( BultCat ).OfClass( typeof( FamilyInstance ) )
.Cast()
.Where( x => x.Symbol.Family.Name.Equals( familyTypeName ) );
}
static IEnumerable Linq2(
Document doc,
BuiltInCategory BultCat,
string familyTypeName )
{
return new FilteredElementCollector( doc ).OfClass( typeof( FamilyInstance ) ).Cast()
.Where( x => x.get_Parameter( BuiltInParameter.ELEM_FAMILY_PARAM )
.AsValueString() == familyTypeName );
}
static IEnumerable FilterRule(
Document doc,
// ElementId ActiveViewId,
BuiltInCategory BultCat,
string familyTypeName )
{
return new FilteredElementCollector( doc )//,ActiveViewId)
.OfCategory( BultCat )
.OfClass( typeof( FamilyInstance ) )
.WherePasses(
new ElementParameterFilter(
new FilterStringRule(
new ParameterValueProvider(
new ElementId( BuiltInParameter.ELEM_FAMILY_PARAM ) ),
new FilterStringEquals(), familyTypeName, true ) ) );
}
static IEnumerable Factory(
Document doc,
BuiltInCategory BultCat,
string familyTypeName )
{
return new FilteredElementCollector( doc )
.OfCategory( BultCat )
.OfClass( typeof( FamilyInstance ) )
.WherePasses(
new ElementParameterFilter(
ParameterFilterRuleFactory.CreateEqualsRule(
new ElementId( BuiltInParameter.ELEM_FAMILY_PARAM ), familyTypeName, true ) ) );
}
#endregion // Retrieve named family symbols using either LINQ or a parameter filter
#region Timer
public class Timer
{
[DllImport( "Kernel32.dll" )]
private static extern bool QueryPerformanceCounter(
out long lpPerformanceCount );
[DllImport( "Kernel32.dll" )]
private static extern bool QueryPerformanceFrequency(
out long lpFrequency );
private long startTime, stopTime;
private long freq;
///
/// Constructor
/// summary>
public Timer()
{
startTime = 0;
stopTime = 0;
if( !QueryPerformanceFrequency( out freq ) )
{
throw new Win32Exception(
"high-performance counter not supported" );
}
}
///
/// Start the timer
/// summary>
public void Start()
{
Thread.Sleep( 0 ); // let waiting threads work
QueryPerformanceCounter( out startTime );
}
///
///Stop the timer
/// summary>
public void Stop()
{
QueryPerformanceCounter( out stopTime );
}
///
/// Return the duration of the timer in seconds
/// summary>
public double Duration
{
get
{
return (double) ( stopTime - startTime )
/ (double) freq;
}
}
}
#endregion // Timer
```
Many thanks to Sudhan for implementing this benchmark and reporting these useful (and reassuring) results!
I created a complete Visual Studio solution and added the missing bits and pieces to test this live in
the [FilterBenchmark GitHub repository](https://github.com/jeremytammik/FilterBenchmark).
I hope that this encourages you to do some benchmarking as well and helps you optimise your own filtered element collectors.