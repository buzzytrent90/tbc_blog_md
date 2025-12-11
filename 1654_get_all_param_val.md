---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 10.2
content_type: qa
optimization_date: '2025-12-11T11:44:16.469880'
original_url: https://thebuildingcoder.typepad.com/blog/1654_get_all_param_val.html
post_number: '1654'
reading_time_minutes: 10
series: general
slug: get_all_param_val
source_file: 1654_get_all_param_val.md
tags:
- doors
- elements
- filtering
- levels
- parameters
- revit-api
- rooms
- sheets
- transactions
- views
- walls
- windows
title: Get All Param Val
word_count: 2071
---

### Getting All Parameter Values
People sometimes ask how to export all the Revit data to an external database.
Obviously, a huge amount of data is embedded in BIM specific constraints and relationships that are hard to extract.
It is very simple to extract all the parameter data, though, as you can see by looking at the numerous existing samples that do so.
Let's implement yet another one ourselves:
- [Existing sample implementations](#2)
- [Black box input](#3)
- [Choices for the output and its structure](#4)
- [Retrieve parameter values from an element](#5)
- [FilterCategoryRule versus category filters](#6)
- [Category description extension method](#7)
- [Retrieve parameter data for all elements of given categories](#8)
- [External command `Execute` mainline](#9)
- [Sample run results](#10)
- [Download](#11)
#### Existing Sample Implementations
In fact, this kind of data access and parameter value extraction is achieved by one of the very
first [ADN Revit API training labs](https://github.com/jeremytammik/AdnRevitApiLabsXtra),
the external
command [Lab4_2_ExportParametersToExcel in Labs4.cs](https://github.com/jeremytammik/AdnRevitApiLabsXtra/blob/master/XtraCs/Labs4.cs#L268-L509),
which is very similar to the official Revit SDK ArchSample.
It creates a dictionary of all elements with a valid category and sorts them by category name.
Next, for each category, it determines a list of all parameters attached to any one of the elements.
Finally, for each element, it reads all its parameter values and exports them to Excel.
In the process, it creates an Excel workbook to store the data, and, for each category, a worksheet to store all the element data.
Here are some discussions of it:
- [Exporting parameter data to Excel, and re-importing](http://thebuildingcoder.typepad.com/blog/2012/09/exporting-parameter-data-to-excel.html)
- [ArchSample and retrieving element properties](http://thebuildingcoder.typepad.com/blog/2015/06/archsample-active-transaction-and-adnrme-for-revit-mep-2016.html#2)
Another sample that goes about it even more professionally is RDBLink. It was originally included in the Revit SDK sample collection, and later removed to be maintained as a separate proprietary subscription product:
- [Integration with a database or ERP system](http://thebuildingcoder.typepad.com/blog/2009/07/integration-with-a-database-or-erp-system.html)
- [Adding a column to RDBLink export](http://thebuildingcoder.typepad.com/blog/2009/11/adding-a-column-to-rdblink-export.html)
- [Parameter access and scheduling](http://thebuildingcoder.typepad.com/blog/2010/05/parameter-access-and-scheduling.html)
- [ODBC export](http://thebuildingcoder.typepad.com/blog/2012/11/survey-and-project-base-point.html#3)
- [RDBLink and exporting data from Revit](http://thebuildingcoder.typepad.com/blog/2016/02/reorg-fomt-devcon-ted-qr-custom-exporter-quality.html#6)
When an RVT file is translated by [Forge](https://autodesk-forge.github.io), the process also captures all the BIM element parameters.
Since it is easy to modify them and add new ones in the Forge viewer, I implemented an add-in to support a full read-write round-trip
workflow, [RvtMetaProp](https://github.com/jeremytammik/rvtmetaprop):
- [Forge meta property editor and RvtMetaProp Revit add-in – executive summary](http://thebuildingcoder.typepad.com/blog/2017/10/rational-bim-programming-at-au-darmstadt.html#5.5)
- [Use Forge or spreadsheet to create shared parameters](http://thebuildingcoder.typepad.com/blog/2017/09/use-forge-or-spreadsheet-to-create-shared-parameters.html)
Today, I thought I would isolate the most basic and generic functionality conceivable to support this kind of workflow, by implementing a simple black box that takes a very specific input and returns a specific output for that:
- Input: a list of BIM categories
- Output: for each category, a dictionary mapping all the elements of that category to a collection of all their parameter values.
#### Black Box Input
The list of categories can simply consist of an array of built-in categories like this:
```csharp
///
/// List all built-in categories of interest
/// summary>
static BuiltInCategory[] _cats =
{
BuiltInCategory.OST_Doors,
BuiltInCategory.OST_Rooms,
BuiltInCategory.OST_Windows
};
```
Defining the output and its structure is more challenging and depends on the exact requirements.
#### Choices for the Output and its Structure
As simple as this sounds, there are quite a couple of choices to be made:
- Parameter identification: name? multiple names?
- Parameter value storage: string representation, or underlying database value?
- Element parameters to retrieve: `Parameters` property, `GetOrderedParameters` method, etc.
In order to simplify things, the parameter values could all be returned as strings.
In the simplest solution, the display strings shown in the Revit user interface, returned by the `AsValueString` method.
A more complex solution might return their real underlying database values instead.
Ideally, the parameters could be identified by their name.
In that case, in the return value could be structured like this, as a dictionary mapping category names to dictionaries mapping element unique ids to dictionaries mapping each elements parameter name to the corresponding value:
```csharp
Dictionary>>
map_cat_to_uid_to_param_values;
```
Unfortunately, though, parameter names are not guaranteed to be unique.
Therefore, it may be impossible to include all parameter values in a dictionary using the parameter name as a key.
Therefore, I resorted to the simple solution of returning a list of strings instead, where each string is formatted by separating the parameter name and the value by an equal sign '=' like this:
```csharp
param_values.Add( string.Format( "{0}={1}",
p.Definition.Name, p.AsValueString() ) );
```
That leads to the following structure for the return value:
```csharp
Dictionary>>
map_cat_to_uid_to_param_values;
```
Finally, we have several different possibilities to retrieve the parameters from the element.
Two obvious choices are to use the `Element` `Parameters` property that retrieves a set containing all the parameters.
Another one offered by the API is the `GetOrderedParameters` method that gets the visible parameters in the order they appear in the UI.
Finally, you can attempt to retrieve values for all the built-in parameters; this approach is used by
the [RevitLookup snooping tool](https://github.com/jeremytammik/RevitLookup) and
the [BipChecker](https://github.com/jeremytammik/BipChecker) built-in parameter checker.
#### Retrieve Parameter Values from an Element
Based on the choices described above, and opting for the simplest solution, we retrieve the parameter values from a given element like this:
```csharp
///
/// Return all the parameter values
/// deemed relevant for the given element
/// in string form.
/// summary>
List GetParamValues( Element e )
{
// Two choices:
// Element.Parameters property -- Retrieves
// a set containing all the parameters.
// GetOrderedParameters method -- Gets the
// visible parameters in order.
IList ps = e.GetOrderedParameters();
List param_values = new List(
ps.Count );
foreach( Parameter p in ps)
{
// AsValueString displays the value as the
// user sees it. In some cases, the underlying
// database value returned by AsInteger, AsDouble,
// etc., may be more relevant.
param_values.Add( string.Format( "{0}={1}",
p.Definition.Name, p.AsValueString() ) );
}
return param_values;
}
```
#### FilterCategoryRule versus Category Filters
Before we can retrieve the parameter data from the elements, we need to retrieve the elements from the Revit database.
As always, this is achieved using a filtered element collector.
Just last week,
we [clarified the use of the `FilterCategoryRule` class](http://thebuildingcoder.typepad.com/blog/2018/05/how-to-use-filtercategoryrule.html).
That discussion led me to believe that it could be used to achieve exactly what I need, filtering for all elements belonging to a given list of categories.
I implemented that code, and it does not seem to do what I expect at all.
In fact, the following code appears to be returning all elements, including those with null categories:
```csharp
List ids
= new List( cats )
.ConvertAll( c
=> new ElementId( (int) c ) );
FilterCategoryRule r
= new FilterCategoryRule( ids );
ElementParameterFilter f
= new ElementParameterFilter( r, true );
// Run the collector
FilteredElementCollector els
= new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.WhereElementIsViewIndependent()
.WherePasses( f );
```
In the end, I resorted to using my tried and proven technique using a logical `OR` of individual category filters instead, like this:
```csharp
// Use a logical OR of category filters
IList a
= new List( cats.Length );
foreach( BuiltInCategory bic in cats )
{
a.Add( new ElementCategoryFilter( bic ) );
}
LogicalOrFilter categoryFilter
= new LogicalOrFilter( a );
// Run the collector
FilteredElementCollector els
= new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.WhereElementIsViewIndependent()
.WherePasses( categoryFilter );
```
It is shorter, and, above all, it works!
#### Category Description Extension Method
Finally, in order to define the key string for the category dictionary, I implemented a little helper method on the built-in category enum:
```csharp
public static class JtBuiltInCategoryExtensionMethods
{
///
/// Return a descriptive string for a built-in
/// category by removing the trailing plural 's'
/// and the OST_ prefix.
/// summary>
public static string Description(
this BuiltInCategory bic )
{
string s = bic.ToString().ToLower();
s = s.Substring( 4 );
Debug.Assert( s.EndsWith( "s" ), "expected plural suffix 's'" );
s = s.Substring( 0, s.Length - 1 );
return s;
}
}
```
As explained in the comment, it returns a descriptive string for a built-in category enumeration value by removing its trailing plural 's' and 'OST_' prefix.
#### Retrieve Parameter Data for all Elements of Given Categories
With these helper methods in place, we can retrieve the parameter data for all elements of a given list categories like this:
```csharp
///
/// Return parameter data for all
/// elements of all the given categories
/// summary>
Dictionary>>
GetParamValuesForCats(
Document doc,
BuiltInCategory[] cats )
{
// Set up the return value dictionary
Dictionary>>
map_cat_to_uid_to_param_values
= new Dictionary>>();
// One top level dictionary per category
foreach( BuiltInCategory cat in cats )
{
map_cat_to_uid_to_param_values.Add(
cat.Description(),
new Dictionary>() );
}
// Collect all required elements
// The FilterCategoryRule as used here seems to
// have no filtering effect at all!
// It passes every single element, afaict.
List ids
= new List( cats )
.ConvertAll( c
=> new ElementId( (int) c ) );
FilterCategoryRule r
= new FilterCategoryRule( ids );
ElementParameterFilter f
= new ElementParameterFilter( r, true );
// Use a logical OR of category filters
IList a
= new List( cats.Length );
foreach( BuiltInCategory bic in cats )
{
a.Add( new ElementCategoryFilter( bic ) );
}
LogicalOrFilter categoryFilter
= new LogicalOrFilter( a );
// Run the collector
FilteredElementCollector els
= new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.WhereElementIsViewIndependent()
.WherePasses( categoryFilter );
// Retrieve parameter data for each element
foreach( Element e in els )
{
Category cat = e.Category;
if( null == cat )
{
Debug.Print(
"element {0} {1} has null category",
e.Id, e.Name );
continue;
}
List param_values = GetParamValues( e );
BuiltInCategory bic = (BuiltInCategory)
(e.Category.Id.IntegerValue);
string catkey = bic.Description();
string uid = e.UniqueId;
map_cat_to_uid_to_param_values[catkey].Add(
uid, param_values );
}
return map_cat_to_uid_to_param_values;
}
```
#### External Command Execute Mainline
The code to run and test this is trivial:
```csharp
Dictionary>>
map_cat_to_uid_to_param_values;
#endif // PARAMETER_NAMES_ARE_UNIQUE
map_cat_to_uid_to_param_values
= GetParamValuesForCats( doc, _cats );
```
Displaying some relevant portion of the results takes much more:
```csharp
List keys = new List(
map_cat_to_uid_to_param_values.Keys );
keys.Sort();
foreach( string key in keys )
{
Dictionary> els
= map_cat_to_uid_to_param_values[key];
int n = els.Count;
Debug.Print( "{0} ({1} element{2}){3}",
key, n, Util.PluralSuffix( n ),
Util.DotOrColon( n ) );
if( 0 < n )
{
List uids = new List( els.Keys );
string uid = uids[0];
List param_values = els[uid];
param_values.Sort();
n = param_values.Count;
Debug.Print( " first element {0} has {1} parameter{2}{3}",
uid, n, Util.PluralSuffix( n ),
Util.DotOrColon( n ) );
param_values.ForEach( pv
=> Debug.Print( " " + pv ) );
}
}
```
#### Sample Run Results
In my test run, I am interested only in door, room and window categories.
From the results, I just display the numbers of elements retrieved, and the parameter values for one single sample element.
Running it in the Revit basic sample project displays the following in the Visual Studio debug output window:

```
door (16 elements):
  first element 59371552-5800-43eb-9ba3-609565158fc5-00067242 has 11 parameters:
    Comments =
    Finish =
    Frame Material =
    Frame Type =
    Head Height = 2100
    Image =
    Level = Level 1
    Mark =
    Phase Created = Working Drawings
    Phase Demolished = None
    Sill Height = 0
room (14 elements):
  first element e6ac360b-aaed-4c3b-a130-36b4c2ac9d13-000d1467 has 21 parameters:
    Area = 27 m²
    Base Finish =
    Base Offset = 0
    Ceiling Finish =
    Comments =
    Computation Height = 0
    Department =
    Floor Finish =
    Image =
    Level = Level 2
    Limit Offset = 6500
    Name =
    Number =
    Occupancy =
    Occupant =
    Perimeter = 29060
    Phase = Working Drawings
    Unbounded Height = 6500
    Upper Limit = Level 2
    Volume = 118.32 m³
    Wall Finish =
window (17 elements):
  first element 6cbabf1d-e8d0-47f0-ac4d-9a7923128d37-0006fb07 has 22 parameters:
    Bottom Hung Casement = No
    Casement Pivot = No
    Casement Swing in Plan = No
    Casement = SH_Aluminum, Anodized Black
    Comments =
    Frame = SH_Aluminum, Anodized Black
    Glass =
    Head Height = 2700
    Height = 2700
    Image =
    Install Depth (from outside) = 80
    Level = Level 2
    Mark =
    Phase Created = Working Drawings
    Phase Demolished = None
    Rough Height = 2700
    Rough Width = 1500
    Sill Height = 0
    Top Hung Casement = Yes
    Width = 1500
    Window Cill Exterior = SH_Aluminum, Anodized Black
    Window Cill Interior = Wood_Walnut black
```

#### Download
I added this code to the
new [module CmdParamValuesForCats.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdParamValuesForCats.cs)
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2019.0.140.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.140.0).
I hope you find it useful.
![Forge meta property editor](img/meta_editor.png)