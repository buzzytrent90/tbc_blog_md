---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.1
content_type: qa
optimization_date: '2025-12-11T11:44:17.026346'
original_url: https://thebuildingcoder.typepad.com/blog/1911_adjacent_wall_room.html
post_number: '1911'
reading_time_minutes: 7
series: general
slug: adjacent_wall_room
source_file: 1911_adjacent_wall_room.md
tags:
- csharp
- elements
- filtering
- levels
- parameters
- revit-api
- rooms
- sheets
- transactions
- vbnet
- views
- walls
- windows
title: Adjacent Wall Room
word_count: 1482
---

### Installer, List of Failures, Adjacent Rooms and Walls
We discuss enhancements to RevitLookup, a list of all built-in Revit failures, and a neat utility to determine all room-wall adjacencies:
- [Adjacent rooms and walls](#2)
- [List of all built-in failures](#3)
- [Recent RevitLookup updates](#4)
- [RevitLookup installation](#5)
![Wall adjacent rooms](img/wall_adjacent_rooms.jpg "Wall adjacent rooms")
#### Adjacent Rooms and Walls
The [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) question
on how to [extract the names of the rooms separated by a wall](https://forums.autodesk.com/t5/revit-api-forum/extract-the-names-of-the-rooms-separated-by-a-wall/m-p/10428696)
prompted Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas
to share a neat utility method implementing much more than that:
\*\*Question:\*\* I use Revit to model my projects, then I export the BOMs to a spreadsheet that creates my quotes for clients.
However, I am running into a problem:
When I need to demolish or create a wall, I would like to see its location, if any, appear in the BOM.
For example, if I break a wall between the kitchen and the living room, I would like to see "kitchen" or "living room", or even both, next to the surfaces. But I can't find a way to implement this.
Do you have any ideas to help me?
\*\*Answer:\*\* Via the API, you can use
the [`Room.GetBoundarySegments` method](https://www.revitapidocs.com/2022/8e0919af-6172-9d16-26d2-268e42f7e936.htm).
This will return a nested list of `BoundarySegment` objects on which you can query their `ElementId` or `LinkElementId` property.
Then it is just a case or matching up the ElementId with the wall id and pairing that with the room(s).
Some walls will be part of more than two rooms:
```vbnet
Private Function Obj_210629a( _
ByVal commandData As Autodesk.Revit.UI.ExternalCommandData, _
ByRef message As String, _
ByVal elements As Autodesk.Revit.DB.ElementSet) _
As Result
Dim UIDoc As UIDocument = commandData.Application.ActiveUIDocument
If UIDoc Is Nothing Then Return Result.Cancelled Else
Dim IntDoc As Document = UIDoc.Document
Dim FEC As New FilteredElementCollector(IntDoc)
Dim ECF As New ElementCategoryFilter(BuiltInCategory.OST_Rooms)
Dim Els As List(Of ElementId) = FEC.WherePasses(ECF) _
.WhereElementIsNotElementType.ToElementIds
Dim WallRefs As New Dictionary(Of ElementId, List(Of String))
For i = 0 To Els.Count - 1
Dim Rm As Room = IntDoc.GetElement(Els(i))
Dim BSegs As IList(Of IList(Of BoundarySegment)) = _
Rm.GetBoundarySegments(New SpatialElementBoundaryOptions)
For x = 0 To BSegs.Count - 1
Dim BSegs0 As IList(Of BoundarySegment) = BSegs(x)
For y = 0 To BSegs0.Count - 1
Dim Bseg As BoundarySegment = BSegs0(y)
If Bseg.ElementId <> ElementId.InvalidElementId Then
Dim CurRef As List(Of String)
If WallRefs.ContainsKey(Bseg.ElementId) Then
CurRef = WallRefs(Bseg.ElementId)
Else
CurRef = New List(Of String)
WallRefs.Add(Bseg.ElementId, CurRef)
End If
Dim Rm_el As Element = Rm
Dim RoomName As String = Rm_el.Name
If CurRef.Contains(RoomName) = False Then
CurRef.Add(RoomName)
End If
End If
Next
Next
Next
Dim FECw As New FilteredElementCollector(IntDoc)
Dim ECFw As New ElementClassFilter(GetType(Wall))
Dim Elsw As List(Of ElementId) = FECw.WherePasses(ECFw).ToElementIds
Using Tx As New Transaction(IntDoc, "Name walls based on rooms")
If Tx.Start = TransactionStatus.Started Then
For i = 0 To Elsw.Count - 1
'Some boundary segments will not relate to walls.
If WallRefs.ContainsKey(Elsw(i)) Then
Dim El As Element = IntDoc.GetElement(Elsw(i))
'Using instance comments parameter for convenience
Dim P As Parameter = El.Parameter(BuiltInParameter.ALL_MODEL_INSTANCE_COMMENTS)
If P Is Nothing Then Continue For Else
Dim Refs As List(Of String) = WallRefs(Elsw(i))
Refs.Sort()
Dim V As New Text.StringBuilder
For j = 0 To Refs.Count - 1
If j = Refs.Count - 1 Then
V.Append(Refs(j))
Else
V.Append(Refs(j) & " / ")
End If
Next
P.Set(V.ToString)
End If
Next
Tx.Commit()
End If
End Using
Return Result.Succeeded
End Function
```
Thanks to Richard for sharing this very nice VB.NET implementation!
Spelling it out, this does two things:
- For all rooms, for each of its bounding walls, note the room to wall relationship
- Use this to create a dictionary mapping the wall element id to a list of the rooms it bounds
- For each wall element id in the dictionary, add a list of the names of the rooms it bounds to its comment field
I ported the VB.NET code to C# and [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples/compare/2022.0.150.14...2022.0.150.15):
```csharp
///
/// For all rooms, determine all adjacent walls,
/// create dictionary mapping walls to adjacent rooms,
/// and tag the walls with the adjacent room names.
/// summary>
void TagWallsWithAdjacentRooms( Document doc )
{
FilteredElementCollector rooms
= new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.OfCategory( BuiltInCategory.OST_Rooms );
Dictionary>> map_wall_to_rooms
= new Dictionary>();
SpatialElementBoundaryOptions opts
= new SpatialElementBoundaryOptions();
foreach( Room room in rooms )
{
IList> loops
= room.GetBoundarySegments( opts );
foreach (IList loop in loops )
{
foreach( BoundarySegment seg in loop )
{
ElementId idWall = seg.ElementId;
if( ElementId.InvalidElementId != idWall )
{
if(!map_wall_to_rooms.ContainsKey(idWall))
{
map_wall_to_rooms.Add(
idWall, new List() );
}
string room_name = room.Name;
if(!map_wall_to_rooms[idWall].Contains( room_name ) )
{
map_wall_to_rooms[ idWall ].Add( room_name );
}
}
}
}
}
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Add list of adjacent rooms to wall comments" );
Dictionary>.KeyCollection ids
= map_wall_to_rooms.Keys;
foreach( ElementId id in ids )
{
Element wall = doc.GetElement( id );
Parameter p = wall.get_Parameter(
BuiltInParameter.ALL_MODEL_INSTANCE_COMMENTS );
if( null != p )
{
string s = string.Join( " / ",
map_wall_to_rooms[ id ] );
p.Set( s );
}
}
tx.Commit();
}
}
```
Note that the C# version processes all elements whose element id appears as a key in the wall to room mapping, regardless of whether they are in fact a wall or not.
The VB.NET version uses a filtered element collector to retrieve only wall elements.
You need to decide which approach better matches your specific requirements.
This is yet another example of a relationship inversion:
every room maintains a relationship to its bounding elements, the walls.
by retrieving and processing that mapping, we can invert the relationship and use that to add information to each wall about its adjacent rooms.
This is a common Revit API task.
A similar [relationship inverter](http://thebuildingcoder.typepad.com/blog/2008/10/relationship-in.html) was
the topic of one of The Building Coder's very first posts, #16, in October 2008.
#### List of All Built-In Failures
Gábor Schnierer very kindly shares a list of
all [built-in failures](https://www.revitapidocs.com/2021.1/eda15d4a-6b14-ee6b-0c44-6011077e6cfc.htm) in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [get all warnings](https://forums.autodesk.com/t5/revit-api-forum/get-a-list-of-all-the-revit-warnings/m-p/10399203):
> Using the awesome code
from [@FAIR59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518)
and [@perry.swoboda](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/7186046),
here is
a [spreadsheet containing the BuiltInFailures for Revit 2022](https://docs.google.com/spreadsheets/d/12glULCZL_yJkq7ko_vI-gEHu69dUIoiCRnmvdLpIoSU/edit#gid=0) with
their `Severity`, `Classname`, `Guid` and `Description`.
Might come handy.
Thank you very much, Gábor!
#### Recent RevitLookup Updates
The number of pull requests to add enhancements
to [RevitLookup](https://github.com/jeremytammik/RevitLookup) has
increased recently significantly.
That is great news!
Each individual improvement may be small and simple.
However, they all add up, and the entire community ends up enjoying a brilliant and full-fledged tool.
Here are the important enhancements made since
the [previous bunch of updates](https://thebuildingcoder.typepad.com/blog/2021/05/revitlookup-update-fuslogvw-and-override-joins.html).
- [2022.0.0.10](https://github.com/jeremytammik/RevitLookup/releases/tag/2022.0.0.10) fix error where element cannot be retrieved for an element id because `SupportedColorFillCategoryIds` returns category ids instead
- [2022.0.0.11](https://github.com/jeremytammik/RevitLookup/releases/tag/2022.0.0.11) add `PlanViewRange` functionality to display view range level id and offset
- [2022.0.0.13](https://github.com/jeremytammik/RevitLookup/releases/tag/2022.0.0.13) add `OnLoad` to increase and optimise width of Snoop window value `ListView` last column
- [2022.0.0.15](https://github.com/jeremytammik/RevitLookup/releases/tag/2022.0.0.15) add RevitLookup.Installation
Many thanks to
all [contributors](https://github.com/jeremytammik/RevitLookup/graphs/contributors) for
your great support!
#### RevitLookup Installation
Luiz Henrique [@ricaun](https://github.com/ricaun) Cassettari created
the [RevitLookup.Installation](https://github.com/ricaun/RevitLookup.Installation) project,
a simple installation using [Inno Setup](https://jrsoftware.org/isinfo.php) to
extract the files to the `ApplicationPlugins` folder.
It generates a digitally signed version of RevitLookup and includes multi-version support for the Revit releases 2017, 2018, 2019, 2020, 2021 and 2022.
It can obviously also be used as a starting point for your own add-in installer.
Many thanks to Luiz Henrique for this and his other nice contributions!