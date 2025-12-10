---
post_number: "1755"
title: "Room Volume Ds"
slug: "room_volume_ds"
author: "Jeremy Tammik"
tags: ['csharp', 'elements', 'family', 'filtering', 'geometry', 'parameters', 'references', 'revit-api', 'rooms', 'sheets', 'transactions', 'views', 'walls', 'windows']
source_file: "1755_room_volume_ds.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1755_room_volume_ds.html"
---

### DirectShape Element to Represent Room Volume
Yesterday, I implemented a new add-in, RoomVolumeDirectShape, that creates `DirectShape` elements representing the volumes of all the rooms.
I'll also mention some challenges encountered en route, some free add-ins shared by Cherry BIM Services, an insight in the meaning of the MEP fitting `Loss Method` property, and AI-generated talking head models:
- [Request to display room volumes in Forge SVF file](#2)
- [RoomVolumeDirectShape functionality](#3)
- [Retrieving all element properties](#4)
- [Converting a .NET dictionary to JSON](#5)
- [Generating `DirectShape` from `ClosedShell`](#6)
- [Complete external command class `Execute` method](#7)
- [Sample model and results](#8)
- [Challenges encountered underway](#9)
- [Licensing system error 22](#9.1)
- [Valid direct shape categories](#9.2)
- [Direct shape phase and visibility](#9.3)
- [Cherry BIM Services](#10)
- [On the value of the "Loss Method" property](#11)
- [AI-generated talking head models](#12)
#### Request to Display Room Volumes in Forge SVF File
The [RoomVolumeDirectShape add-in](https://github.com/jeremytammik/RoomVolumeDirectShape) was
inspired by the following request:
The context: We are building digital twins out of BIM data. To do so, we use Revit, Dynamo, and Forge.
The issue: We rely on the rooms in Revit to perform a bunch of tasks (reassign equipment localization, rebuild a navigation tree, and so on).
Unfortunately, these rooms are not displayed in the Revit 3D view.
Therefore, they are nowhere to be found in the Forge SVF file.
Our (so-so) solution: uses Dynamo to extract the room geometry and build Revit volumes.
It works, but it is:
- Not very robust: Some rooms has to be recreated manually, Dynamo crashes, geometry with invalid faces is produced, etc.
- Not very fast: The actual script exports SAT files and reimports them.
- Manual: Obviously, and also tedious and error-prone.
The whole process amounts to several hours of manual work.
We want to fix this.
Our goal: A robust implementation that will get rid of Dynamo, automate the process in Revit, and in the end, run that in a Forge Design Automation process.
The ideal way forward is exactly what you describe: A native C# Revit API that find the rooms, creates a direct shape volume for them, and copy their properties to that.
No intermediate formats, no UI, just straight automation work.
#### RoomVolumeDirectShape Functionality
Fulfilling this request, I implemented a
new [RoomVolumeDirectShape add-in](https://github.com/jeremytammik/RoomVolumeDirectShape) that
performs the following simple steps:
- Retrieve all rooms in the BIM using a filtered element collector
- For each room:
- Query the room for its closed shell using
the [ClosedShell API call](https://www.revitapidocs.com/2020/1a510aef-63f6-4d32-c0ff-a8071f5e23b8.htm)
- Generate a [DirectShape element](https://www.revitapidocs.com/2020/bfbd137b-c2c2-71bb-6f4a-992d0dcf6ea8.htm) representing the room volume geometry
- Query the room for all its properties, stored in parameters
(cf., [getting all parameter values](https://thebuildingcoder.typepad.com/blog/2018/05/getting-all-parameter-values.html)
and [retrieving parameter values from an element](https://thebuildingcoder.typepad.com/blog/2018/05/getting-all-parameter-values.html#5))
- Generate a JSON string representing a dictionary of the room properties
- Store the room property JSON string in the `DirectShape` element `Comment` property
#### Retrieving All Element Properties
The `GetParamValues` method retrieves and returns all the element parameter values in a dictionary mapping parameter names to the corresponding values.
For each entry, it also appends a single-character indicator of the parameter storage type to the key.
It makes use of two helper methods:
- `ParameterStorageTypeChar`, to return a key character for each storage type
- `ParameterToString`, to retrieve the parameter value as a string
```csharp
///
/// Return parameter storage type abbreviation
/// summary>
static char ParameterStorageTypeChar(
Parameter p )
{
if( null == p )
{
throw new ArgumentNullException(
"p", "expected non-null parameter" );
}
char abbreviation = '?';
switch( p.StorageType )
{
case StorageType.Double:
abbreviation = 'r'; // real number
break;
case StorageType.Integer:
abbreviation = 'n'; // integer number
break;
case StorageType.String:
abbreviation = 's'; // string
break;
case StorageType.ElementId:
abbreviation = 'e'; // element id
break;
case StorageType.None:
throw new ArgumentOutOfRangeException(
"p", "expected valid parameter "
+ "storage type, not 'None'" );
}
return abbreviation;
}
///
/// Return parameter value formatted as string
/// summary>
static string ParameterToString( Parameter p )
{
string s = "null";
if( p != null )
{
switch( p.StorageType )
{
case StorageType.Double:
s = p.AsDouble().ToString( "0.##" );
break;
case StorageType.Integer:
s = p.AsInteger().ToString();
break;
case StorageType.String:
s = p.AsString();
break;
case StorageType.ElementId:
s = p.AsElementId().IntegerValue.ToString();
break;
case StorageType.None:
s = "none";
break;
}
}
return s;
}
///
/// Return all the element parameter values in a
/// dictionary mapping parameter names to values
/// summary>
static Dictionary GetParamValues(
Element e )
{
// Two choices:
// Element.Parameters property -- Retrieves
// a set containing all the parameters.
// GetOrderedParameters method -- Gets the
// visible parameters in order.
//IList ps = e.GetOrderedParameters();
ParameterSet pset = e.Parameters;
Dictionary d
= new Dictionary( pset.Size );
foreach( Parameter p in pset )
{
// AsValueString displays the value as the
// user sees it. In some cases, the underlying
// database value returned by AsInteger, AsDouble,
// etc., may be more relevant, as done by
// ParameterToString
string key = string.Format( "{0}({1})",
p.Definition.Name,
ParameterStorageTypeChar( p ) );
string val = ParameterToString( p );
if( d.ContainsKey( key ) )
{
if( d[key] != val )
{
d[key] += " | " + val;
}
}
else
{
d.Add( key, val );
}
}
return d;
}
```
#### Converting a .NET Dictionary to JSON
`FormatDictAsJson` converts the .NET dictionary of element properties to a JSON-formatted string:
```csharp
///
/// Return a JSON string representing a dictionary
/// mapping string key to string value.
/// summary>
static string FormatDictAsJson(
Dictionary d )
{
List keys = new List( d.Keys );
keys.Sort();
List key_vals = new List(
keys.Count );
foreach( string key in keys )
{
key_vals.Add(
string.Format( "\"{0}\" : \"{1}\"",
key, d[key] ) );
}
return "{" + string.Join( ", ", key_vals ) + "}";
}
```
#### Generating DirectShape from ClosedShell
With the element parameter property retrieval and JSON formatting helper methods in place, very little remains to be done.
We gather all the rooms in the BIM using a filtered element collector, aware of the fact that the `Room` class only exists in the Revit API, not internally in Revit.
The filtered element collector therefore has to retrieve `SpatialElement` objects instead and use .NET post-processing to extract the rooms,
cf. [accessing room data](http://thebuildingcoder.typepad.com/blog/2011/11/accessing-room-data.html).
Once we have the rooms, we can process each one as follows:
- Retrieve room volume from `ClosedShell`
- Retrieve room properties
- Format properties into JSON string
- Create direct shape
- Set its geometry to the room volume
- Set its application data id to the room's `UniqueId`
- Set its name to contain the room name
- Store the room property dictionary in its comment parameter
```csharp
GeometryElement geo = r.ClosedShell;
Dictionary param_values
= GetParamValues( r );
string json = FormatDictAsJson( param_values );
DirectShape ds = DirectShape.CreateElement(
doc, _id_category_for_direct_shape );
ds.ApplicationId = id_addin;
ds.ApplicationDataId = r.UniqueId;
ds.SetShape( geo.ToList() );
ds.get_Parameter( _bip_properties ).Set( json );
ds.Name = "Room volume for " + r.Name;
```
#### Complete External Command Class Execute Method
For the sake of completeness, here is the entire external command class and execute method implementation:
```csharp
#region Namespaces
using System;
using System.Linq;
using System.Collections.Generic;
using System.Diagnostics;
using Autodesk.Revit.ApplicationServices;
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Architecture;
using Autodesk.Revit.UI;
#endregion
namespace RoomVolumeDirectShape
{
[Transaction( TransactionMode.Manual )]
public class Command : IExternalCommand
{
// Cannot use OST_Rooms; DirectShape.CreateElement
// throws ArgumentExceptionL: Element id categoryId
// may not be used as a DirectShape category.
///
/// Category assigned to the room volume direct shape
/// summary>
ElementId _id_category_for_direct_shape
= new ElementId( BuiltInCategory.OST_GenericModel );
///
/// DirectShape parameter to populate with JSON
/// dictionary containing all room properies
/// summary>
BuiltInParameter _bip_properties
= BuiltInParameter.ALL_MODEL_INSTANCE_COMMENTS;
// ... Property retrieval and JSON formatting helper methods ...
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiapp = commandData.Application;
UIDocument uidoc = uiapp.ActiveUIDocument;
Application app = uiapp.Application;
Document doc = uidoc.Document;
string id_addin = uiapp.ActiveAddInId.ToString();
IEnumerable rooms
= new FilteredElementCollector( doc )
.WhereElementIsNotElementType()
.OfClass( typeof( SpatialElement ) )
.Where( e => e.GetType() == typeof( Room ) )
.Cast();
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Generate Direct Shape Elements "
+ "Representing Room Volumes" );
foreach( Room r in rooms )
{
Debug.Print( r.Name );
GeometryElement geo = r.ClosedShell;
Dictionary param_values
= GetParamValues( r );
string json = FormatDictAsJson( param_values );
DirectShape ds = DirectShape.CreateElement(
doc, _id_category_for_direct_shape );
ds.ApplicationId = id_addin;
ds.ApplicationDataId = r.UniqueId;
ds.SetShape( geo.ToList() );
ds.get_Parameter( _bip_properties ).Set( json );
ds.Name = "Room volume for " + r.Name;
}
tx.Commit();
}
return Result.Succeeded;
}
}
}
```
For the full Visual Studio solution and updates to the code, please refer to
The [RoomVolumeDirectShape GitHub repository](https://github.com/jeremytammik/RoomVolumeDirectShape).
#### Sample Model and Results
I tested this in the standard Revit \*rac_basic_sample_project.rvt\* sample model:
![Revit Architecture rac_basic_sample_project.rvt](img/rac_basic_sample_project.png)
Isolated, the resulting direct shapes look like this:
![DirectShape elements representing room volumes](img/rac_basic_sample_project_room_volumes.png)
#### Challenges Encountered Underway
I ran into a couple of issues en route that cost me time to resolve, ever though absolutely trivial, so I'll make a note of them here for my own future reference:
- [Licensing system error 22](#9.1)
- [Valid direct shape categories](#9.2)
- [Direct shape phase and visibility](#9.3)
#### Licensing System Error 22
Something happened on my virtual Windows machine, and I saw an error saying:

```
  ---------------------------
  Autodesk Revit 2020
  ---------------------------
  Licensing System Error 22
  Failed to locate Adls
  ---------------------------
  OK
  ---------------------------
```

Luckily, a similar issue has already been discussed in the forum thread
on [licensing system error 22 – failed to locate `Adls`](https://forums.autodesk.com/t5/installation-licensing/error-de-sistema-de-licencias-22-failed-to-locate-adls/td-p/8771037).
The solution described there worked fine in my case as well:
- Run Services.msc
- Check the entry for Autodesk Desktop Licensing Service
- If it is not already running, start the service
#### Valid Direct Shape Categories
I had to fiddle a bit choosing which category to use for the `DirectShape` element creation.
The rooms category is not acceptable, generic model and structural framing is.
Attempting to use an invalid category throws an ArgumentException saying, \*Element id categoryId may not be used as a DirectShape category.\*
#### Direct Shape Phase and Visibility
Right away after the first trial run, I could see the resulting `DirectShape` elements in RevitLookup, and all their properties looked fine.
However, try as I might, I was unable to see them in the Revit 3D view...
...until I finally flipped through the phases and found the right one.
The model is apparently in a state in which newly created geometry lands in a phase that is not displayed in the default 3D view.
#### Cherry BIM Services
Enough on my activities.
Someone else has also been pretty active recently:
[Ninh Truong Huu Ha](https://github.com/haninh2612) of [Cherry BIM Services](http://www.cherrybimservices.com) recently
shared several free Revit add-ins, and also published code for one of them.
Oops, the code has disappeared again from Ninh's GitHub repository; in fact, the whole repository disappeared...
> Inspired by Jeremy Tammik and Harry Mattison who always share their incredible knowledge to the world, I decided from now on, all of my Revit add-ins will be free to use for all Revit users.
One year ago, I had absolutely zero knowledge of the coding world, e.g., C#, Revit API, Visual Studio, etc.
I would never have thought that someday I could have my own Revit add-in published in the Autodesk Store.
- Start from my first add-in: [Batch Rename Revit Type name with Naming convention](https://www.dropbox.com/sh/fs1b60jewyfkdxd/AAArHy7C6Y7edtBGckl2AIeSa?dl=0).
Here is a three-minute [demonstration video](https://youtu.be/n91iyjOALdo).
- [Warning Manager by Cherry BIM Services](https://apps.autodesk.com/RVT/en/Detail/Index?id=7980350830610368901&appLang=en&os=Win64)
- [Auto-generate curtain grids](https://youtu.be/Sacd3K6RBbU) – [Auto curtain wall Dropbox download](https://www.dropbox.com/sh/rfllne68zjjjq9t/AAA7eLI-p1LqFHkRj3fBlxpza?dl=0)
- [Batch Upgrade models, templates and families](https://youtu.be/rciLWaik2_0)
Many thanks to Ninh for sharing these tools!
#### On the Value of the 'Loss Method' Property
Next, let's point out an MEP analysis related question raised and solved by Hanley Deng in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [how to get the value of the property 'Loss Method'](https://forums.autodesk.com/t5/revit-api-forum/how-to-get-the-value-of-the-property-quot-loss-method-quot/m-p/8816013):
\*\*Question:\*\* Pipe fittings have a property named "Loss Method".
In the UI, its value is "Use Definition on Type".
In the API, however, the value is a GUID, e.g., "3bf616f9-6b98-4a21-80ff-da1120c8f6d6":
![Loss method parameter property](img/snoop_loss_method_param_val.png)
How can I convert the API GUID value, "3bf616f9-6b98-4a21-80ff-da1120c8f6d6", into the UI value, "Use Definition on Type"?
\*\*Answer:\*\* The loss method can be programmed, so the GUID you see might be something like the add-in identifier, c.f. this discussion on
the [pipe fitting K factor](https://thebuildingcoder.typepad.com/blog/2017/12/pipe-fitting-k-factor-archilab-and-installer.html).
\*\*Response:\*\* Problem solved. This problem is solved in 2 cases:
1. For Pipe fittings, when Loss Method is "Use definition on Type":
In this case, the `parameter.AsString()` value equals the GUID stored in Autodesk.Revit.DB.MEPCalculatationServerInfo.PipeUseDefinitionOnTypeGUID.
In this case, I cannot find the UI display string for it, so I hardcode the UI display string.
2. I all other cases, including other values in Pipe Fittings, and all the values in Duct Fittings, the `ServerName` is the string in the UI display, accessible through the following API call:
```csharp
Autodesk.Revit.DB.MEPCalculatationServerInfo
.GetMEPCalculationServerInfo(objFamilyInstance),
```
Many thanks to Hanley for clarifying this.
#### AI-Generated Talking Head Models
Finally, let's close with this impressive demonstration of AI simulated talking head models, presented in the five-minute video
on [few-shot adversarial learning of realistic neural talking head models](https://youtu.be/p1b5aiTrGzY):