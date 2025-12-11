---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.1
content_type: qa
optimization_date: '2025-12-11T11:44:16.182294'
original_url: https://thebuildingcoder.typepad.com/blog/1507_parameter_definition.html
post_number: '1507'
reading_time_minutes: 3
series: parameters
slug: parameter_definition
source_file: 1507_parameter_definition.md
tags:
- elements
- family
- parameters
- revit-api
- sheets
- views
- walls
title: Parameter Definition
word_count: 615
---

### Parameter Definition Overview
We have repeatedly discussed all kinds of different aspects
of [Revit element parameters](http://thebuildingcoder.typepad.com/blog/parameters),
but not put together
a [topic group](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5) for them yet.
Today I am happy to present a pretty comprehensive overview and explanation of the process of defining a shared parameter by none less than Scott Conover himself, Senior Revit Engineering Manager:
\*\*Question:\*\* What do I need to do to programmatically create a shared parameter?
I would like to set the `SetAllowVaryBetweenGroups` flag on it.
![New shared parameter](img/new_shared_parameter.jpg)
\*\*Answer:\*\* You create the details needed to define a shared parameter from `ExternalDefinition`.
Existing shared parameter file entries can be read to become an `ExternalDefinition` in your code, or you can create a new entry in the current shared parameter file using the `DefinitionGroup.Create` method.
The sample code listed in the Revit API help file `RevitAPI.chm` or the online Revit API docs
under [InstanceBinding class](http://www.revitapidocs.com/2017/7978cb57-0a48-489e-2c8f-116fa2561437.htm) shows
this process best.
Here is part of that sample snippet:
```csharp
public bool SetNewParameterToInstanceWall(
UIApplication app,
DefinitionFile myDefinitionFile )
{
// Create a new group in the shared parameters file
DefinitionGroups myGroups = myDefinitionFile.Groups;
DefinitionGroup myGroup = myGroups.Create( "MyParameters" );
// Create an instance definition in definition group MyParameters
ExternalDefinitionCreationOptions option
= new ExternalDefinitionCreationOptions(
"Instance_ProductDate", ParameterType.Text );
// Don't let the user modify the value, only the API
option.UserModifiable = false;
// Set tooltip
option.Description = "Wall product date";
Definition myDefinition_ProductDate
= myGroup.Definitions.Create( option );
. . .
```
The return from `DefinitionGroup.Create` is an `ExternalDefinition`, even though the type declared is the parent class.
Once you have an `ExternalDefinition`, you add it to the document.
There are several ways:
1. Use `InstanceBinding` as shown in that sample.
2. Use `FamilyManager.AddParameter` to add the parameter to a family.
3. Use `FamilyManager.ReplaceParameter` to replace a family parameter with the shared one.
4. Use `SharedParameterElement.Create` to create the element that represents the parameter without binding it to any categories.
There are also some `RebarShape` related utilities which I would not recommend for general usage but might be OK for rebar-specific code.
Once the parameter is in the document, it has an `InternalDefinition`.
The best ways to get it:
1. If you have the `ParameterElement` from #4 you can use `ParameterElement.GetDefinition`.
2. If you have the GUID (which you should, since it is provided by the `ExternalDefinition`), you can use `SharedParameterElement.Lookup` followed by `ParameterElement.GetDefinition`.
3. If you have an instance of an element whose category has this parameter bound, get the `Parameter` and use `Parameter.Definition`.
Once you have the `InternalDefinition`, you can access the vary across groups option as well as other things.
You can also use an `InternalDefintion` for adding and removing `InstanceBindings` to categories.
Many thanks to Scott for this nice comprehensive summary and overview!
#### Addemdum
Joshua Lumley pointed out some possible enhancements
in [his two](http://thebuildingcoder.typepad.com/blog/2016/12/parameter-definition-overview.html#comment-3079825547)
[comments](http://thebuildingcoder.typepad.com/blog/2016/12/parameter-definition-overview.html#comment-3079829813) below:
To run the code more than twice I added:
```csharp
bool dgMatchFound = false;
foreach( DefinitionGroup dg in myGroups )
{
if( dg.Name == myGroupName )
{
dgMatchFound = true;
myGroup = dg;
}
}
if( dgMatchFound == false )
{
myGroup = myGroups.Create( myGroupName );
}
```
and
```csharp
bool dMatchFound = false;
foreach( Definition d in myGroup.Definitions )
{
if( d.Name == newParameterName )
{
dMatchFound = true;
myDefinition_ProductDate = d;
}
}
if( !dMatchFound )
{
myDefinition_ProductDate
= myGroup.Definitions.Create( option );
}
```
I called it like this:
```csharp
DefinitionFile defFile
= GetOrCreateSharedParamsFile(
ActiveUIDocument.Application.Application );
bool AddParameterResult
= SetNewParameterToInstanceWall(
ActiveUIDocument.Application, defFile );
TaskDialog.Show( "Did it work",
AddParameterResult.ToString() );
```
Many thanks to Josh for the helpful usage hints!