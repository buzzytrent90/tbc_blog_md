---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.9
content_type: qa
optimization_date: '2025-12-11T11:44:16.994459'
original_url: https://thebuildingcoder.typepad.com/blog/1898_2022_job_column_inst.html
post_number: '1898'
reading_time_minutes: 10
series: structural
slug: 2022_job_column_inst
source_file: 1898_2022_job_column_inst.md
tags:
- csharp
- elements
- family
- filtering
- parameters
- references
- revit-api
- sheets
- transactions
- vbnet
- views
- structural
title: 2022 Job Column Inst
word_count: 1970
---

### Sneak Peek, Automatic Columns, Arcs Angles and Careers
A new version of Revit coming soon, solutions for automatic column type creation and instance placement, handling arc angles, career opportunities and a sustainable economic model:
- [What's new in Revit 2022 sneak peek](#2)
- [Creating column types from list](#3)
- [Normalising arc start and end angle](#4)
- [Many exciting opportunities at Autodesk](#5)
- [The sustainable doughnut economic model](#6)
#### What's New in Revit 2022 Sneak Peek
Are you interested in the new features and enhancements in the upcoming next release of Revit?
Take the opportunity to join the \*sneak peek on What's New in Revit 2022\* later today,
on [April 6, 2021, at 21:30 CET](https://www.timeanddate.com/worldclock/converter.html?iso=20210406T193000&p1=268):
- [What's New in Revit 2022 Sneak Peek](https://youtu.be/FjSbv6W6tcg)
Then, register to join the Autodesk product experts for the full \*What's New in Revit 2022\* webinar next week,
[April 13, 2021, at 10:00am PT, 1:00pm ET](https://www.timeanddate.com/worldclock/converter.html?iso=20210413T170000&p1=268&p2=tz_pt&p3=tz_et).
You can register here:
- [Link to register for the full \*What's New in Revit 2022\* webinar on April 13](https://autode.sk/31xUn2g)
Here are some other forward-looking Revit resources:
- [Revit Public Roadmap](https://trello.com/b/ldRXK9Gw/revit-public-roadmap)
- [Application to the Revit Community and the Revit Preview to try new features in advance](https://feedback.autodesk.com/key/LHMJFVHGJK085G2M)
- [Join the Autodesk Product Research Community](https://www.autodeskproductresearch.com/hub)
![Revit 2022 sneak peek](img/2021-04-06_rvt_2022_sneak_peek.png "Revit 2022 sneak peek")
#### Creating Column Types from List
Richard [RPThomas108](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1035859) Thomas provided
a nice complete solution to create all required column types from a list of rectangular width and height dimensions in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [create columns types](https://forums.autodesk.com/t5/revit-api-forum/create-columns-types/m-p/10181049):
\*\*Question:\*\* I have the family \*M_Concrete-Rectangular-Column\* and two lists of doubles that represent `b` and `h` of the column.
How can I extract the unique values of the two lists and then create new family types using them?
\*\*Answer:\*\* Here is a solution in VB.NET:
```vbnet
Private Class ColumnType
Inherits EqualityComparer(Of ColumnType)
Private IntD As Integer() = New Integer(1) {}
Public ReadOnly Property H As Integer
Get
Return IntD(0)
End Get
End Property
Public ReadOnly Property W As Integer
Get
Return IntD(1)
End Get
End Property
Public ReadOnly Property Name As String
Get
Return CStr(H) & "x" & CStr(W)
End Get
End Property
Public Sub New(D1 As Integer, D2 As Integer)
If D1 > D2 Then
IntD = New Integer() {D1, D2}
Else
IntD = New Integer() {D2, D1}
End If
End Sub
Public Overrides Function Equals(x As ColumnType, y As ColumnType) As Boolean
Return x.H = y.H AndAlso x.W = y.W
End Function
Public Overrides Function GetHashCode(obj As ColumnType) As Integer
Return obj.Name.GetHashCode
End Function
End Class
Private Function TObj168(ByVal commandData As Autodesk.Revit.UI.ExternalCommandData,
ByRef message As String, ByVal elements As Autodesk.Revit.DB.ElementSet) As Result
Dim UIDoc As UIDocument = commandData.Application.ActiveUIDocument
If UIDoc Is Nothing Then Return Result.Cancelled Else
Dim IntDoc As Document = UIDoc.Document
Dim L1 As Double() = New Double() {100, 200, 150, 500, 400, 300, 250, 250}
Dim L2 As Double() = New Double() {200, 200, 150, 500, 400, 300, 250, 250}
Dim all As New List(Of ColumnType)
For i = 0 To L1.Length - 1
all.Add(New ColumnType(L1(i), L2(i)))
Next
all = all.Distinct(New ColumnType(0, 0)).ToList
Dim FEC As New FilteredElementCollector(IntDoc)
Dim ECF As New ElementCategoryFilter(BuiltInCategory.OST_StructuralColumns)
Dim Els As List(Of FamilySymbol) = FEC.WherePasses(ECF).WhereElementIsElementType.Cast(Of FamilySymbol).ToList
'Use column name here
Dim Existing As List(Of FamilySymbol) = Els.FindAll(Function(x) x.FamilyName = "Concrete-Rectangular-Column")
If Existing.Count = 0 Then
Return Result.Cancelled
End If
Dim AlreadyExists As New List(Of ColumnType)
Dim ToBeMade As New List(Of ColumnType)
For i = 0 To all.Count - 1
Dim Ix As Integer = i
Dim J As FamilySymbol = Existing.Find(Function(x) x.Name = all(Ix).Name)
If J Is Nothing Then
ToBeMade.Add(all(i))
Else
AlreadyExists.Add(all(i))
End If
Next
If ToBeMade.Count = 0 Then
Return Result.Cancelled
End If
Using Tr As New Transaction(IntDoc, "Make types")
If Tr.Start = TransactionStatus.Started Then
For i = 0 To ToBeMade.Count - 1
Dim Itm = ToBeMade(i)
Dim Et As ElementType = Existing(0).Duplicate(Itm.Name)
'Use actual type parameter names
'Use GUIDs instead of LookupParameter where possible
Et.LookupParameter("h")?.Set(304.8 \* Itm.H)
Et.LookupParameter("b")?.Set(304.8 \* Itm.W)
Next
Tr.Commit()
End If
End Using
Return Result.Succeeded
End Function
```
I converted it to C#
and [added it to The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples/compare/2021.0.150.21...2021.0.150.22) for
future reference:
```csharp
class ColumnType : EqualityComparer
{
int[] _dim = new int[ 2 ];
public int H
{
get { return _dim[ 0 ]; }
}
public int W
{
get { return _dim[ 1 ]; }
}
public string Name
{
get { return H.ToString() + "x" + W.ToString(); }
}
public ColumnType( int d1, int d2 )
{
if( d1 > d2 )
{
_dim = new int[] { d1, d2 };
}
else
{
_dim = new int[] { d2, d1 };
}
}
public override bool Equals( ColumnType x, ColumnType y )
{
return x.H == y.H && x.W == y.W;
}
public override int GetHashCode( ColumnType obj )
{
return obj.Name.GetHashCode();
}
}
Result CreateColumnTypes( Document doc )
{
int[] L1 = new int[] { 100, 200, 150, 500, 400, 300, 250, 250 };
int[] L2 = new int[] { 200, 200, 150, 500, 400, 300, 250, 250 };
List all = new List();
for( int i = 0; i < L1.Length; ++i )
{
all.Add( new ColumnType( L1[ i ], L2[ i ] ) );
}
all = all.Distinct( new ColumnType( 0, 0 ) ).ToList();
FilteredElementCollector symbols
= new FilteredElementCollector( doc )
.WhereElementIsElementType()
.OfCategory( BuiltInCategory.OST_StructuralColumns );
// Column name to use
string column_name = "Concrete-Rectangular-Column";
IEnumerable existing
= symbols.Cast()
.Where( x
=> x.FamilyName.Equals( column_name ) );
if( 0 == existing.Count() )
{
return Result.Cancelled;
}
List AlreadyExists = new List();
List ToBeMade = new List();
for( int i = 0; i < all.Count ; ++i )
{
FamilySymbol fs = existing.FirstOrDefault(
x => x.Name == all[ i ].Name );
if( fs == null )
{
ToBeMade.Add( all[ i ] );
}
else
{
AlreadyExists.Add( all[ i ] );
}
}
if( ToBeMade.Count == 0 )
{
return Result.Cancelled;
}
using( Transaction tx = new Transaction( doc ) )
{
if( tx.Start( "Make types" ) == TransactionStatus.Started )
{
FamilySymbol first = existing.First();
foreach( ColumnType ct in ToBeMade )
{
ElementType et = first.Duplicate( ct.Name );
// Use actual type parameter names
// Use GUIDs instead of LookupParameter where possible
et.LookupParameter( "h" ).Set( 304.8 \* ct.H );
et.LookupParameter( "b" ).Set( 304.8 \* ct.W );
}
tx.Commit();
}
}
return Result.Succeeded;
}
```
An interesting aspect here is the notion of units.
If we were comparing internal units (fractions of feet), we could not have used the integer for comparison.
However, since we assume it is mm and column sizes are rounded, we can use them easily.
So, if we take the smallest measurement of length, then we usually have less problems.
In an imperial example, if we had used inches, then we can compare an integer of 6 inches rather than a double 0.5 ft.
I guess if it is 6.5 inches we are stuck.
However, the mm is the smallest unit we generally work with (unless we are talking paint), so can represent smaller fractions of something as whole numbers.
It is what it is.
It would be interesting to explore intelligent rounding and unit dependent real-number fuzz.
One can well consider 1 mm the minimal metric unit in Revit, since it does not (or hardly) support smaller lengths.
For imperial sizes, I guess I would go for 1/16 of an inch.
Depending on what units the model happens to prefer, one could select one or the other dynamically and implement an rounding algorithm that adapts accordingly.
Talking about the creation of columns and column types, let's add a pointer to another related thread
on [automatic column creation from imported CAD drawing](https://forums.autodesk.com/t5/revit-api-forum/automatic-column-creation-from-imported-cad-drawing/m-p/9648240):
![Place columns from imported 2D CAD](img/place_columns_from_cad_dwg_3.png "Place columns from imported 2D CAD")
#### Normalising Arc Start and End Angle
Richard also shared some useful notes on normalising arc start and end angles in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to retrieve startAngle and endAngle of arc object](https://forums.autodesk.com/t5/revit-api-forum/how-to-retrieve-startangle-and-endangle-of-arc-object/m-p/10213537):
What I've noticed recently is that the actual parameters of an arc can be far greater than 2 \* pi, so how does this happen?
If you draw a detail arc and drag the ends of the arc one at a time around the circumference, the parameter values will accumulate and become greater than 2 pi (after 1 cycle).
They don't reset; you can effectively wind up the arc parameters this way (I discovered parameter numbers reaching 1000 degrees).
We could not create arcs otherwise since the creation method enforces p0 must be less than p1 and arcs are drawn anti clockwise.
When you cross the 2 pi or 0 boundary, you need p1 not to be starting again from 0, otherwise it would be less than p0 on the other side of the boundary.
So, as @JimJia noted, they are not reliable for arc start end angles (or are if the ends have not been manipulated in such a way).
However, in a way, you can get back to the correct angles between 0 and 2 x pi (since each rotation is a multiple of 2 x pi) i.e.
- Divide by 2 \* pi
- Deduct integer part of result
- If this is less than 0 add 1 (to negative number)
- Multiply by 2 \* pi
I'm not sure why you would need to do this however.
Other thing I would note is always use `Arc.XDirection` and `Arc.Normal` with `AngleOnPlaneTo` (rather than assuming), since arc can be flipped or rotated.
Also, above was related to winding the arc ends around the circumference in an anti-clockwise direction to increase parameter values.
If user drags ends around in a clockwise direction, then you get negative values for the parameters.
Takes less movement from original by user to go into negative domain, so this is more likely to be seen perhaps.
Many thanks to Richard for these helpful observations!
#### Many Exciting Opportunities at Autodesk
Autodesk is offering a many exciting career opportunities in Europe and elsewhere right now:
- Software Engineering Manager – Krakow, Poland – Job ID 21WD47267
- Construction Account Executive, Territory – Sweden, remote – Job ID 21WD47481
- Territory Account Executive, Construction – The Netherlands – Job ID 21WD47524
- Strategic Account Manager Manufacturing – Munich, Germany or home office – Job ID 21WD47310
- Principal Software Engineer – Cambridge, UK – Job ID 21WD45889
Feel free to ask me for a referral.
Good luck applying for one of them and others all over the world in
the [Autodesk career site](https://www.autodesk.com/careers)!
#### The Sustainable Doughnut Economic Model
The [doughnut economic model](https://en.wikipedia.org/wiki/Doughnut_(economic_model)) is
gaining official acceptance.
For instance, [Amsterdam bet its post-Covid recovery on doughnut economics, and more cities are now following suit](https://www.cnbc.com/2021/03/25/amsterdam-brussels-bet-on-doughnut-economics-amid-covid-crisis.html).
> The Doughnut, or Doughnut economics, is a visual framework for sustainable development – shaped like a doughnut or lifebelt – combining the concept of planetary boundaries with the complementary concept of social boundaries:
![Doughnut economic model](img/doughnut_economic_model.jpg "Doughnut economic model")