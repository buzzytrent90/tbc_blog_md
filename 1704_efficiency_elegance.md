---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.7
content_type: qa
optimization_date: '2025-12-11T11:44:16.564445'
original_url: https://thebuildingcoder.typepad.com/blog/1704_efficiency_elegance.html
post_number: '1704'
reading_time_minutes: 6
series: general
slug: efficiency_elegance
source_file: 1704_efficiency_elegance.md
tags:
- doors
- elements
- family
- filtering
- parameters
- revit-api
- schedules
- sheets
- transactions
- walls
title: Efficiency Elegance
word_count: 1226
---

### Efficient and Elegant Code
I remain busy, mainly in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160).
Here are two recent samples dealing with pretty generic efficiency related questions:
- [Efficiency and elegance in simple code](#2)
- [Pushing wall type to doors](#3)
![Elegance and efficiency](img/1704_elegance_efficiency.png)
#### Efficiency and Elegance in Simple Code
One recent topic in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) asks
for [help with efficiency and elegance in simple code](https://forums.autodesk.com/t5/revit-api-forum/need-help-with-efficiency-and-elegance-in-simple-code/m-p/8410687):
\*\*Question:\*\* My code works, but I don't believe it's as efficient as it could be.
Specifically, the Try/catch area of the code.
If I don't try/catch each line separately, it misses some of the doors.
Like to reduce the regeneration time too if possible.
Any pointers are appreciated.
Thanks.
```csharp
// Get doors
FilteredElementCollector collector = new FilteredElementCollector( doc );
List coll = collector.OfClass( typeof( FamilyInstance ) )
.OfCategory( BuiltInCategory.OST_Doors )
.ToList();
// Filtered element collector is iterable
foreach( Element e in coll )
{
// Get the parameter name
Parameter s_parameter = e.LookupParameter( "Swing Angle" );
Parameter s1_parameter = e.LookupParameter( "Swing Angle_Door 1" );
Parameter s2_parameter = e.LookupParameter( "Swing Angle_Door 2" );
using( Transaction t = new Transaction( doc, "parameters" ) )
// Modify document within a transaction
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Change door swing angles to 45" );
try
{
s_parameter.Set( 0.785398163 );
}
catch { }
try
{
s1_parameter.Set( 0.785398163 );
}
catch { }
try
{
s2_parameter.Set( 0.785398163 );
}
catch { }
tx.Commit();
}
}
TaskDialog.Show( "Completed", "Door swings changed to 45°." );
return Result.Succeeded;
```
\*\*Answer by MarryTookMyCoffe:\*\*
- Change `List` to `Array`; you don't have to change `foreach` to `for`, because the compiler usually does it for you, but remember that without optimization of code, `for` is faster than `foreach`.
- Delete the first
```csharp
using (Transaction t = new Transaction(doc, "parameters"))
```
- Why is this even there? You put `using` transaction inside `using` transaction; why?
- Put the loop inside transaction, not outside (every start and commit takes a lot of time).
- If you get parameter with string property, you have to check if parameter is not null:
```csharp
if(s_parameter != null)
```
- Check if the parameter is read-only and if it takes double as value.
- So, application can only return `Succeeded`.
- Are you changing types or instances? You can add `WhereElementIsElementType` to the filtered element collector.
Try this:
```csharp
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
private const double angle = 0.785398163;
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
Document doc = commandData.Application.ActiveUIDocument.Document;
// Get instance of doors
Element[] coll = new FilteredElementCollector( doc )
.OfClass( typeof( FamilyInstance ) )
.OfCategory( BuiltInCategory.OST_Doors )
.WhereElementIsNotElementType()
.ToArray();
// Start transaction outside of loop, that way you only open transaction once
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Change door swing angles to 45" );
foreach( Element e in coll )
{
//set Parameter by the name
try
{
SetParameter( e, "Swing Angle", angle );
SetParameter( e, "Swing Angle_Door 1", angle );
SetParameter( e, "Swing Angle_Door 2", angle );
}
catch { }
}
tx.Commit();
}
TaskDialog.Show( "Completed", "Door swings changed to 45°." );
return Result.Succeeded;
}
///
/// set double parameter by value
/// summary>
/// param>
/// param>
/// param>
/// returns>
private static bool SetParameter( Element element, string Name, double value )
{
Parameter parameter = element.LookupParameter( Name );
// Preventing exceptions is better than catching it
if( parameter != null && !parameter.IsReadOnly && parameter.StorageType == StorageType.Double )
return parameter.Set( value );
return false;
}
}
```
\*\*Response:\*\* Thanks, works perfectly. Now I'll step through the code so I can learn what you did. Thanks Again!
\*\*Answer 2:\*\* I would like to add a few points more:
`LookupParameter` will only retrieve the first parameter of the given name. In order to use it, you must be absolutely certain that only one parameter with the given name exists. I would suggest adding a test somewhere in your add-in startup code to ensure that this really is the case.
It is always safer to use other means to retrieve parameters, e.g., the built-in parameter enumeration value, if it exists, or simply the `Parameter` definition, which you can look up and cache beforehand, e.g., in the afore-mentioned code ensuring that the parameter name on that element type really is unique.
There is no need to convert the element collection to an array, i.e., saying
```csharp
Element[] coll = new FilteredElementCollector( doc )...ToArray();
```
You can perfectly well leave it as a filtered element collector and iterate directly over that instead:
```csharp
FilteredElementCollector coll = new FilteredElementCollector( doc )...;
foreach( Element e in coll ) ...
```
That will save conversion time and space and avoid the unnecessary data duplication, cf. the discussions
on:
- [`FindElement` and collector optimisation](http://thebuildingcoder.typepad.com/blog/2012/09/findelement-and-collector-optimisation.html)
- [Filtering for family instances and types by family name](https://thebuildingcoder.typepad.com/blog/2017/08/forge-installed-version-move-group-filter-by-name.html#7)
- [Getting the area scheme from an area](https://thebuildingcoder.typepad.com/blog/2017/03/q4r4-first-queries-revitlookup-and-areas-in-schemes.html#5)
#### Pushing Wall Type to Doors
Some pretty similar code appeared in the question
on [pushing wall type to doors](https://forums.autodesk.com/t5/revit-api-forum/push-walltype-to-doors/m-p/8411858):
\*\*Question:\*\* I have been working on this macro for a while, we are looking for a way to have our door schedule to include the wall type (available in the Type Mark parameter). I have attached where I am at. I had no issues with pulling an instance parameter in the wall to the instance parameter in the door, but I was not having much luck with the type parameter in the wall. The only area I am unsure of is if I am accessing the type parameters from the wall correctly:
```csharp
public void WalltoDoor2()
{
Document doc = this.ActiveUIDocument.Document;
ElementCategoryFilter hostFilter
= new ElementCategoryFilter(
BuiltInCategory.OST_Walls );
ElementCategoryFilter hostedFilter
= new ElementCategoryFilter(
BuiltInCategory.OST_Doors );
string parameterName = "Type Mark";
string parameterName2 = "Wall Type";
using( Transaction t = new Transaction(
doc, "Set hosted parameters" ) )
{
try
{
t.Start();
foreach( Element host in
new FilteredElementCollector( doc )
.WherePasses( hostFilter ) )
{
if( host != null )
{
Element hostT = host.Document.GetElement(
host.GetTypeId() );
Parameter paramHost = hostT.LookupParameter(
parameterName );
foreach( Element hosted in
new FilteredElementCollector( doc )
.WherePasses( hostedFilter )
.OfClass( typeof( FamilyInstance ) )
.Cast()
.Where( q => q.Host.Id == host.Id ) )
{
if( hosted != null )
{
hosted.LookupParameter( parameterName2 )
.Set( paramHost.AsString() );
}
}
}
}
t.Commit();
}
catch( Exception )
{
}
}
}
```
\*\*Answer:\*\* Using the built-in parameter checker, it looks like the Type Mark is called "ALL_MODEL_TYPE_MARK" for a wall.
Try using:
```csharp
hostT.LookupParameter("ALL_MODEL_TYPE_MARK");
```
Here are two further suggestions to clean up your code:
- Use `using` for transactions
– [encapsulating a transaction in a `using` statement](https://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html) automagically disposes of it and rolls back if needed.
- Don't catch all exceptions
– [never catch all exceptions](https://thebuildingcoder.typepad.com/blog/2017/05/prompt-cancel-throws-exception-in-revit-2018.html#5), only the ones that you really can handle.
Are you really ready to handle the exceptions 'computer on fire', 'building collapsed', etc.?
If you catch all exceptions, you are preventing the really competent instances from even seeing them.
Cutting the phone lines to the fire brigade and police, so to speak.
You may be endangering your computer and your valued person.
Well, at least your valued Revit model.