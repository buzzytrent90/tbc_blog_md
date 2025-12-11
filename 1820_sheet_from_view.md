---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 6.1
content_type: qa
optimization_date: '2025-12-11T11:44:16.811630'
original_url: https://thebuildingcoder.typepad.com/blog/1820_sheet_from_view.html
post_number: '1820'
reading_time_minutes: 7
series: views
slug: sheet_from_view
source_file: 1820_sheet_from_view.md
tags:
- elements
- family
- filtering
- parameters
- references
- revit-api
- schedules
- selection
- sheets
- views
title: Sheet From View
word_count: 1450
---

### Getting Title Block Data and ViewSheet from View
Today, let's highlight two view related data access topics:
- [Get ViewSheet from View](#2)
- [Title block data access](#3)
#### Get ViewSheet from View
An interesting in-depth conversation in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [get ViewSheet from View](https://forums.autodesk.com/t5/revit-api-forum/get-viewsheet-from-view/m-p/7075550) presents
two different approaches to determine relationships between sheets and the views they host:
\*\*Question:\*\* Is there a way to get the `ViewSheet` or `ElementId` of the `ViewSheet` from a `View` that is placed on that sheet?
I can get the sheet name and sheet number from the provided parameters but do not see one for the actual ViewSheet.
Am I missing something?
The only thing I can think of to do without this info is to loop through all ViewSheets to match the one with the same sheet number (since sheet number is required to be unique).
\*\*Answer:\*\* Have you looked at the sample code provided in the [description of the ViewSheet class](http://www.revitapidocs.com/2017/af2ee879-173d-df3a-9793-8d5750a17b49.htm)?
```csharp
ICollection views = viewSheet.GetAllPlacedViews();
message += "\nNumber of views in the sheet : " + views.Count;
```
If all else fails, you could use this relationship and invert it, just as you suggest.
The [View.Title property](http://www.revitapidocs.com/2017/bfa96650-310e-5385-3a9d-1a1248b623ce.htm) also
generally consists of the view name plus other modifiers, such as the view type, sheet number, area scheme, and/or assembly type, depending on the specifics of the view:
\*\*Response:\*\* You must have misunderstood my question. I don't have the ViewSheet in order to use the method to get its placed views. I have a view, that I know is on a sheet (because its sheet number parameter is not null), but I want to get a direct reference to that sheet that it is placed on. It seems to me that you are still suggesting that I must loop through all ViewSheets to get the View that I'm concerned with. Thanks.
\*\*Answer:\*\* Yes, indeed, that is exactly what I am suggesting.
Get all the view sheets, keep track of them and the views they host, and invert that relationship, as described in one of the very early discussions by The Building Coder back in 2008 on a [relationship inverter](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html).
That gives you a complete dictionary lookup both ways: you can look up the sheet hosting any view and you can look up all the views hosted by any sheet, instantaneously.
It takes a moment to set up the relationships; after that, all data is available and lookup is very fast.
\*\*Other answer:\*\* I maybe have a workaround for this question, something similar I assume.
Select some views in the Project Browser to know if the views are placed on sheet(s).
Then, run a macro or external command to execute something like code below.
If the view is placed on a sheet, it shows the sheet number, sheet name and whatever data you are interested in from the ViewSheet referenced by the views selected in the Project Browser:
```csharp
public void GetViewSheetFromView( UIDocument uidoc )
{
Document doc = uidoc.Document;
string data = "";
ICollection selectedIds
= uidoc.Selection.GetElementIds();
foreach( ElementId selectedid in selectedIds )
{
View e = doc.GetElement( selectedid ) as View;
foreach( View v in new FilteredElementCollector( doc )
.OfClass( typeof( View ) )
.Cast()
.Where( q => q.Id.Equals( e.Id ) ) )
{
string thisSheet = "";
foreach( ViewSheet vs in new FilteredElementCollector( doc )
.OfClass( typeof( ViewSheet ) )
.Cast() )
{
foreach( ElementId eid in vs.GetAllPlacedViews() )
{
View ev = doc.GetElement( eid ) as View;
if( ev.Id == v.Id )
{
thisSheet += vs.SheetNumber
+ " - " + vs.Name + Environment.NewLine;
break;
}
}
}
if( thisSheet != "" )
{
data += v.ViewType + ": " + v.Name + " " + Environment.NewLine
+ thisSheet.TrimEnd( ' ', ',' ) + Environment.NewLine;
}
else
{
data += v.ViewType + ": " + v.Name + " " + Environment.NewLine
+ thisSheet.TrimEnd( ' ', ',' );
data += " NOT ON SHEET " + Environment.NewLine + "\n";
}
}
}
TaskDialog.Show( "View Report", data );
}
```
![Get ViewSheet from View](img/GetViewSheetFromView.png "Get ViewSheet from View")
\*\*Yet another answer:\*\* You can perhaps get VIEWPORT_SHEET_NUMBER from Parameters.
\*\*Yet another answer:\*\* As suggested above, using the BIP for viewport sheet number with an ElementParameterFilter is probably the best approach.
Sheet numbers are unique in each Revit model, so it is safe to search by them and get the right result.
```vbnet
Public Function TObj43(ByVal commandData As ExternalCommandData,
ByRef message As String, ByVal elements As ElementSet) As Result
If commandData.Application.ActiveUIDocument Is Nothing Then Return Result.Cancelled Else
Dim AcView As View = commandData.Application.ActiveUIDocument.ActiveGraphicalView
Dim P_Ns As Parameter = AcView.Parameter(BuiltInParameter.VIEWPORT_SHEET_NUMBER)
If P_Ns Is Nothing Then
GoTo Monday
End If
Dim Txt As String = P_Ns.AsString
If String.IsNullOrEmpty(Txt) Then GoTo Monday Else
Dim SeFR As FilterRule = ParameterFilterRuleFactory.CreateEqualsRule(
New ElementId(BuiltInParameter.SHEET_NUMBER), Txt, True)
Dim PFilt As New ElementParameterFilter(SeFR, False)
Dim FEC As New FilteredElementCollector(AcView.Document)
Dim ECF As New ElementClassFilter(GetType(ViewSheet))
Dim LandF As New LogicalAndFilter(ECF, PFilt)
Dim Els As List(Of Element) = FEC.WherePasses(LandF).ToElements
If Els.Count <> 1 Then
GoTo Monday
Else
Dim TD As New TaskDialog("Was this your sheet...")
TD.MainInstruction = Txt & "-" & Els(0).Name
TD.MainContent = "Note that view types that can appear on " _
+ "multiple sheets (Legends, Images, Schedules etc.) will " _
+ " not be returned by this method."
TD.Show()
GoTo Friday
End If
Monday:
TaskDialog.Show("Something went amiss...",
"We looked hard in many places but were unable to " _
+ "find your sheet on this occasion.")
Return Result.Failed
Friday:
Return Result.Succeeded
End Function
```
\*\*Yet another answer:\*\* Here is a pretty quick way of checking for a specific sheet:
```csharp
private ViewSheet CheckSheet( string _sheetNumber )
{
ParameterValueProvider pvp = new ParameterValueProvider(
new ElementId( BuiltInParameter.SHEET_NUMBER ) );
FilterStringRuleEvaluator fsr = new FilterStringEquals();
FilterRule fRule = new FilterStringRule(
pvp, fsr, _sheetNumber, true );
ElementParameterFilter filter = new ElementParameterFilter( fRule );
if( new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_Sheets )
.WherePasses( filter )
.FirstOrDefault() is ViewSheet vs )
{
return vs;
}
else
{
return null;
}
}
```
\*\*Summary:\*\* I believe this last solution is unbeatable if you are interested in one single lookup.
It uses a parameter filter and the `SHEET_NUMBER` built-in parameter.
Probably, the VB function `TObj43` above is similarly efficient.
`GetViewSheetFromView` demonstrates the lookup of the inverted relationship I described, but just for one single view.
That code could be used to store the entire relationships mapping sheet to hosted views and the inverted one mapping each view to hosting sheets for all views and sheets. That might possibly be more efficient if you frequently need to look up several different view to sheet relationships.
Thank you all for the very illuminating and helpful answers!
#### Title Block Data Access
\*\*Question:\*\* I am searching for some sample code for use in a Design Automation API project.
Rather than reinvent to wheel, I thought you might have a snippet handy showing how to read Revit drawing title block attributes like these:
![Title block data access](img/title_block_data_access.png "Title block data access")
\*\*Answer:\*\* I discussed accessing
the [title block of a sheet](https://thebuildingcoder.typepad.com/blog/2009/11/title-block-of-sheet.html) myself
back in 2009, but that information is rather antiquated now.
The question was also raised and answered in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [TitleBlock](https://forums.autodesk.com/t5/revit-api-forum/titleblock/td-p/3802588),
and that information is perfectly valid.
Ah, I now found a more recent and useful article on how
to [determine sheet size](https://thebuildingcoder.typepad.com/blog/2010/05/determine-sheet-size.html) that
should provide all you need.
The code is included in
[The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[module CmdSheetSize.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdSheetSize.cs).
The title block instances are family instance elements.
You can access the title block element using a filtered element collector, e.g.:
```csharp
FilteredElementCollector title_block_instances
= new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_TitleBlocks )
.OfClass( typeof( FamilyInstance ) );
```
You can loop through these elements and retrieve the required data from their built-in parameters, such as SHEET_NAME, SHEET_NUMBER, SHEET_DRAWN_BY, SHEET_CHECKED_BY etc., like this:
```csharp
foreach( FamilyInstance e in a )
{
p = e.get_Parameter(
BuiltInParameter.SHEET_NUMBER );
Debug.Assert( null != p,
"expected valid sheet number" );
string sheet_number = p.AsString();
p = e.get_Parameter(
BuiltInParameter.SHEET_WIDTH );
Debug.Assert( null != p,
"expected valid sheet width" );
string swidth = p.AsValueString();
double width = p.AsDouble();
. . .
}
```
Als always, you can use [RevitLookup](https://github.com/jeremytammik/RevitLookup) to explore this data interactively yourself in your own model to see which properties are available where and what other title block information may be of interest to your application.