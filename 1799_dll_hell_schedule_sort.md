---
post_number: "1799"
title: "Dll Hell Schedule Sort"
slug: "dll_hell_schedule_sort"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'levels', 'parameters', 'revit-api', 'schedules', 'sheets', 'views']
source_file: "1799_dll_hell_schedule_sort.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1799_dll_hell_schedule_sort.html"
---

### DLL Conflicts and Replicating Schedule Sort Order
As always, I remain busy in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160).
Here, today, I highlight the following topics:
- [I caved in to smartphone](#2)
- [Handling third party library DLL conflicts](#3)
- [Replicating schedule sort order](#4)
#### I Caved in to SmartPhone
After several decades of hard resistance, I finally caved in.
Both my bank and my VPN network managers forced me to set up a smartphone.
I got an old Android S5, reset it to factory settings, installed the updates, and [removed all possible Samsung ballast](https://www.guidingtech.com/29921/remove-samsung-galaxy-s5).
It is working fine now, and I am still resisting all further temptation to go smartphone-crazy, not setting up dozens of apps or anything.
Just switching it on for the login credential checking, and then immediately off again.
Avoiding [smartphone danger](https://duckduckgo.com/?q=smartphone+danger);
[too much digital is bad for you](https://duckduckgo.com/?q=Too+much+digital+is+bad+for+you).
#### Handling Third Party Library DLL Conflicts
Back to a more technology friendly topic... or is it?
One recurring problematic topic is
on [handling third party library DLL conflicts](https://thebuildingcoder.typepad.com/blog/2017/06/handling-third-party-library-dll-conflicts.html).
A new query related to that came up, asking:
\*\*Question:\*\* I recently released a new add-in to my colleagues in the company.
Unfortunately, they report a failure starting it if they are already using another third-party add-in.
It throws the following file load exception:
![File load exception](img/fileloadexception_mvvmlight.png)
What can I do to fix this?
\*\*Answer:\*\* This looks like a conflict between two versions of a .NET assembly DLL that both of the add-ins are dependent on.
The two add-ins require different versions and a conflict ensues.
You are not the first to run into such an issue, as you can see from the list of old discussions
on [handling third party library DLL conflicts](https://thebuildingcoder.typepad.com/blog/2017/06/handling-third-party-library-dll-conflicts.html) from 2017.
For new suggestions and solutions that came up since then, you can search
the [Revit API discussion forum](https://forums.autodesk.com/t5/revit-api-forum/bd-p/160) for
'dll conflict' and study more recent articles here on the blog:
- [Revit automatically initializes CefSharp](#https://thebuildingcoder.typepad.com/blog/2019/04/whats-new-in-the-revit-2020-api.html#4.1.1)
- [CefSharp DLL entanglement solution using IPC](#https://thebuildingcoder.typepad.com/blog/2019/04/set-floor-level-and-use-ipc-for-disentanglement.html#4)
- [External DLL loading](#https://thebuildingcoder.typepad.com/blog/2019/09/extensible-storage-external-event-dll-and-pipe-face.html#4)
As you can see from these examples, you have two choices:
- If Revit and / or several different add-ins use the same DLL, ensure that they all use the same version.
- If this is not possible for some reason, disconnect your usage of the problematic DLL from the Revit add-in `AppDomain` and use some other method to access the functionality you require, e.g., via [inter-process communication IPC](https://en.wikipedia.org/wiki/Inter-process_communication).
In this specific case, it sounds to me as if you are the creator of one add-in, and another add-in uses the same support DLL as you do.
In that case, I suggest you modify your add-in to work with whatever version of the support DLL happens to be loaded.
Alternatively, wrap the support DLL functionality into something that you have control over, so that you can do whatever you like with it.
#### Replicating Schedule Sort Order
Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen comes to the rescue once again, presenting a very comprehensive and well explained solution
to [replicate graphical column schedule sort order with C#](https://forums.autodesk.com/t5/revit-api-forum/replicate-graphical-column-schedule-sort-order-with-c/m-p/9105470) by
implementing a custom `ColumnMarkComparer` class implementing `IComparer`:
\*\*Question:\*\* I am trying to write an add-in with C# to automate the process of sequentially labelling structural columns to match the sequence in the graphical column schedule.
I have written the following code to get all the concrete columns in the project and then create a list of their Column Location Marks, i.e., A(2000)-1(2150), built-in category:
```csharp
ElementCategoryFilter filter
= new ElementCategoryFilter(
BuiltInCategory.OST_StructuralColumns );
StructuralMaterialTypeFilter filter_mat
= new StructuralMaterialTypeFilter(
StructuralMaterialType.Concrete );
IList columns = collector
.WherePasses( filter )
.WherePasses( filter_mat )
.WhereElementIsNotElementType()
.ToElements();
List locations = new List();
List colmarks = new List();
foreach( Element ele in columns )
{
string colmark = ele.get_Parameter(
BuiltInParameter.COLUMN_LOCATION_MARK ).AsString();
colmarks.Add( colmark );
}
colmarks.Sort();
```
My trouble comes at this point.
The `colmarks.Sort` method does not match the order of the columns in the graphical column schedule.
I have also tried sorting the columns by their `XYZ` location which doesn't match the order in the graphical column schedule either.
Can anyone provide any insight into how the graphical column schedule orders the columns and how I can replicate this?
Here is the graphical column schedule sort order I am aiming for:
![Graphical column schedule sort order](img/graphical_column_schedule_sort.png)
I've tried a few different ways which involve creating a list of the parameter as strings and then sorting or using `OrderBy` like this:
```csharp
List sortedElements = columns
.OrderBy( x => x.get_Parameter(
BuiltInParameter.COLUMN_LOCATION_MARK ).AsString() )
.ToList();
```
That does not help:
![Graphical column schedule sort order in C#](img/graphical_column_schedule_sort_cs.png)
I have the same issue in Dynamo too; the order seems to match my code but not the graphical column schedule:
![Graphical column schedule sort order in Dynamo](img/graphical_column_schedule_sort_dynamo.png)
I also read the thread on
[Graphical column schedule in Revit – sort by specified parameter](https://forums.autodesk.com/t5/revit-ideas/graphical-column-schedule-in-revit-sort-by-specified-parameter/idi-p/8190431)
and still hope someone can shed some light on the logic Revit uses to sort the columns in the graphical column schedule so that I can replicate it with my own code in C#.
\*\*Answer:\*\* Implement your own comparison class.
LocationMark syntax:

```
  A(xxxx)-1(yyyy)
```

Sort order:
- Grid sequence A: alphabetic ascending
- Grid sequence 1: alphabetic ascending
- xxxx value: first the positive values from smallest to largest, then negative values from smallest to largest
- yyyy value: first the positive values from smallest to largest, then negative values from smallest to largest
```csharp
List GetSortedColumns( Document doc )
{
List colums
= new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.OfCategory( BuiltInCategory.OST_StructuralColumns )
.OfClass( typeof( FamilyInstance ) )
.Cast()
.ToList();
colums.Sort( new ColumnMarkComparer() );
return colums;
}
```
Extension method helper class:
```csharp
public static class Extensions
{
public static string GetColumnLocationMark(
this FamilyInstance f )
{
Parameter p = f.get_Parameter(
BuiltInParameter.COLUMN_LOCATION_MARK );
return( p == null )
? string.Empty
: p.AsString();
}
}
```
Comparer helper class:
```csharp
public class ColumnMarkComparer : IComparer
{
int IComparer.Compare(
FamilyInstance x,
FamilyInstance y )
{
if( x == null )
{
return y == null ? 0 : -1;
}
if( y == null )
return 1;
string[] mark1 = x.GetColumnLocationMark()
.Split( '(', ')' );
string[] mark2 = y.GetColumnLocationMark()
.Split( '(', ')' );
if( mark1.Length < 4 )
{
return mark2.Length < 4 ? 0 : -1;
}
if( mark2.Length < 4 )
return 1;
// gridsequence A
int res = string.Compare(
mark1[ 0 ], mark2[ 0 ] );
if( res != 0 )
return res;
// gridsequence 1
string m12 = mark1[ 2 ].Remove( 0, 1 );
string m22 = mark2[ 2 ].Remove( 0, 1 );
res = string.Compare( m12, m22 );
if( res != 0 )
return res;
// value xxxx
double d1 = 0;
double d2 = 0;
double.TryParse( mark1[ 1 ], out d1 );
double.TryParse( mark2[ 1 ], out d2 );
if( Math.Round( d1 - d2, 4 ) != 0 )
{
if( d1 < 0 ^ d2 < 0 )
{
return d1 < 0 ? 1 : -1;
}
else
{
return Math.Abs( d1 ) < Math.Abs( d2 )
? -1
: 1;
}
}
// value yyyy
double.TryParse( mark1[ 3 ], out d1 );
double.TryParse( mark2[ 3 ], out d2 );
if( Math.Round( d1 - d2, 4 ) != 0 )
{
if( d1 < 0 ^ d2 < 0 )
{
return d1 < 0 ? 1 : -1;
}
else
{
return Math.Abs( d1 ) < Math.Abs( d2 )
? -1
: 1;
}
}
return 0;
}
}
```
Many thanks to Fair59 for providing yet another effective high-level answer, which also solves a subsequent question
on [sorting scheduled elements based on their sort and group fields](https://forums.autodesk.com/t5/revit-api-forum/sort-scheduled-elements-base-on-it-s-sort-group-fields-to-be/m-p/9142870).
I added his code
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples), cf.
the [diff to the previous release](https://github.com/jeremytammik/the_building_coder_samples/compare/2020.0.147.18...2020.0.147.19).