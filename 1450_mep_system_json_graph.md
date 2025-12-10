---
post_number: "1450"
title: "The Building Coder"
slug: "mep_system_json_graph"
author: "Jeremy Tammik"
tags: ['elements', 'levels', 'parameters', 'revit-api', 'selection', 'sheets', 'transactions', 'views', 'windows']
source_file: "1450_mep_system_json_graph.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1450_mep_system_json_graph.html"
---

### MEP System Structure in Hierarchical JSON Graph
Yesterday, I presented the
new [TraverseAllSystems](https://github.com/jeremytammik/TraverseAllSystems) add-in
to [traverse all MEP system graphs](http://thebuildingcoder.typepad.com/blog/2016/06/traversing-and-exporting-all-mep-system-graphs.html) and
export their connected hierarchical structure to JSON and XML that I am helping
the [USC](http://www.usc.edu) team with here at the San Francisco cloud accelerator.
![San Francisco cloud accelerator](img/2016-06_sf_accelerator.jpg)
I continued with that today, and also integrated a minor enhancement to RevitLookup:
- [TraverseAllSystems updates](#1)
- [Shared parameter creation](#2)
- [Options](#3)
- [Bottom-up JSON structure](#4)
- [Top-down JSON structure](#5)
- [TraversalTree JSON output generator](#6)
- [TreeNode JSON output generator](#7)
- [Download and to do](#8)
- [RevitLookup updates](#9)
#### TraverseAllSystems Updates
The aim of the TraverseAllSystems project is to present the MEP system graphs in a separate tree view panel integrated in
the [Forge viewer](https://developer.autodesk.com/en/docs/viewer/v2/overview) and
hook up the tree view nodes bi-directionally with the 2D and 3D viewer elements.
To achieve that, I implemented a couple of significant enhancements over the simple XML file storage:
- Store the MEP system graph structure in JSON instead of XML
- Implement both bottom-up and top-down storage according to
the [jsTree JSON spec](https://www.jstree.com/docs/json).
- Support both element id and UniqueId node identifiers.
- Store the JSON output in a shared parameter attached to the MEP system element,
so that it is automatically included in the Forge SVF translation generated from the RVT input file.
Here is a list of the update releases so far:
- [2017.0.0.2](https://github.com/jeremytammik/TraverseAllSystems/releases/tag/2017.0.0.2) – implemented visited element dictionary to prevent infinite recursion loop
- [2017.0.0.3](https://github.com/jeremytammik/TraverseAllSystems/releases/tag/2017.0.0.3) – implemented DumpToJson
- [2017.0.0.4](https://github.com/jeremytammik/TraverseAllSystems/releases/tag/2017.0.0.4) – implemented shared parameter creation
- [2017.0.0.5](https://github.com/jeremytammik/TraverseAllSystems/releases/tag/2017.0.0.5) – implemented shared parameter value population, tested and verified graph structure json is written out
- [2017.0.0.6](https://github.com/jeremytammik/TraverseAllSystems/releases/tag/2017.0.0.6) – renamed json text field to name
- [2017.0.0.7](https://github.com/jeremytammik/TraverseAllSystems/releases/tag/2017.0.0.7) – implemented top-down json graph storage
- [2017.0.0.8](https://github.com/jeremytammik/TraverseAllSystems/releases/tag/2017.0.0.8) – automatically create shared parameter, eliminated separate command, wrap json strings in double quotes, validated json output
#### Shared Parameter Creation
I implemented a new `SharedParameterMgr` class to create the shared parameter to store the JSON output in.
This class is based on
the [ExportCncFab](https://github.com/jeremytammik/ExportCncFab)
[ExportParameters.cs module](https://github.com/jeremytammik/ExportCncFab/blob/master/ExportCncFab/ExportParameters.cs).
The shared parameter is automatically created if not already present, as in the following usage example:
```csharp
// Check for shared parameter
// to store graph information.
Definition def = SharedParameterMgr.GetDefinition(
desirableSystems.First() );
if( null == def )
{
SharedParameterMgr.Create( doc );
def = SharedParameterMgr.GetDefinition(
desirableSystems.First() );
if( null == def )
{
message = "Error creating the "
+ "storage shared parameter.";
return Result.Failed;
}
}
```
Here is the `SharedParameterMgr` class implementation:
```csharp
///
/// Shared parameters to keep store MEP system
/// graph structure in JSON strings.
/// summary>
class SharedParameterMgr
{
///
/// Define the user visible shared parameter name.
/// summary>
const string _shared_param_name = "MepSystemGraphJson";
///
/// Return the parameter definition from
/// the given element and parameter name.
/// summary>
public static Definition GetDefinition( Element e )
{
IList ps = e.GetParameters(
_shared_param_name );
int n = ps.Count;
Debug.Assert( 1 >= n,
"expected maximum one shared parameters "
+ "named " + _shared_param_name );
Definition d = ( 0 == n )
? null
: ps[0].Definition;
return d;
}
///
/// Create a new shared parameter definition
/// in the specified grpup.
/// summary>
static Definition CreateNewDefinition(
DefinitionGroup group,
string parameter_name,
ParameterType parameter_type )
{
return group.Definitions.Create(
new ExternalDefinitionCreationOptions(
parameter_name, parameter_type ) );
}
///
/// Create the shared parameter.
/// summary>
public static void Create( Document doc )
{
///
/// Shared parameters filename; used only in case
/// none is set and we need to create the export
/// history shared parameters.
/// summary>
const string _shared_parameters_filename
= "shared_parameters.txt";
const string _definition_group_name
= "TraverseAllSystems";
Application app = doc.Application;
// Retrieve shared parameter file name
string sharedParamsFileName
= app.SharedParametersFilename;
if( null == sharedParamsFileName
|| 0 == sharedParamsFileName.Length )
{
string path = Path.GetTempPath();
path = Path.Combine( path,
_shared_parameters_filename );
StreamWriter stream;
stream = new StreamWriter( path );
stream.Close();
app.SharedParametersFilename = path;
sharedParamsFileName
= app.SharedParametersFilename;
}
// Retrieve shared parameter file object
DefinitionFile f
= app.OpenSharedParameterFile();
using( Transaction t = new Transaction( doc ) )
{
t.Start( "Create TraverseAllSystems "
+ "Shared Parameters" );
// Create the category set for binding
CategorySet catSet = app.Create.NewCategorySet();
Category cat = doc.Settings.Categories.get_Item(
BuiltInCategory.OST_DuctSystem );
catSet.Insert( cat );
cat = doc.Settings.Categories.get_Item(
BuiltInCategory.OST_PipingSystem );
catSet.Insert( cat );
Binding binding = app.Create.NewInstanceBinding(
catSet );
// Retrieve or create shared parameter group
DefinitionGroup group
= f.Groups.get_Item( _definition_group_name )
?? f.Groups.Create( _definition_group_name );
// Retrieve or create the three parameters;
// we could check if they are already bound,
// but it looks like Insert will just ignore
// them in that case.
Definition definition
= group.Definitions.get_Item( _shared_param_name )
?? CreateNewDefinition( group,
_shared_param_name, ParameterType.Text );
doc.ParameterBindings.Insert( definition, binding,
BuiltInParameterGroup.PG_GENERAL );
t.Commit();
}
}
}
```
#### Options
I implemented a new `Options` class to control two settings:
- Use element id or UniqueId for to identify node
- Store JSON graph bottom-up or top-down
The class implementation is short, sweet and trivial:
```csharp
class Options
{
///
/// Store element id or UniqueId in JSON output?
/// summary>
public static bool StoreUniqueId = false;
public static bool StoreElementId = !StoreUniqueId;
///
/// Store parent node id in child, or recursive
/// tree of children in parent?
/// summary>
public static bool StoreJsonGraphBottomUp = false;
public static bool StoreJsonGraphTopDown
= !StoreJsonGraphBottomUp;
}
```
The two bottom-up and top-down JSON storage structures both comply with
the [jsTree JSON spec](https://www.jstree.com/docs/json).
#### Bottom-Up JSON Structure

```
[
  { "id" : "ajson1", "parent" : "#", "text" : "Simple root node" },
  { "id" : "ajson2", "parent" : "#", "text" : "Root node 2" },
  { "id" : "ajson3", "parent" : "ajson2", "text" : "Child 1" },
  { "id" : "ajson4", "parent" : "ajson2", "text" : "Child 2" },
]
```

#### Top-Down JSON Structure

```
{
  id: -1,
  name: 'Root',
  children: [
  {
    id: 0,
    name: 'Mechanical System',
    children: [
    {
      id: 0_1,
      name: 'Child 0_1',
      type: 'window',
      otherField: 'something...',
      children: [
      {
        id: 0_1_1,
        name: 'Grandchild 0_1_1'
      }]
    }, {
      id: 0_2,
      name: 'Child 0_2',
      children: [
      {
        id: 0_2_1,
        name: 'Grandchild 0_2_1'
      }]
    }]
  }, {
    id: 2,
    name: 'Electrical System',
    children: [
    {
      id: 2_1,
      name: 'Child 2_1',
      children: [{
        id: 2_1_1,
        name: 'Grandchild 2_1_1'
      }]
    },
    {
      id: 2_2,
      name: 'Child 2_2',
      children: [{
        id: 2_2_1,
        name: 'Grandchild 2_2_1'
      }]
    }]
  },
  {
    id: 3,
    name: 'Piping System',
    children: [
    {
      id: 3_1,
      name: 'Child 3_1',
      children: [{
        id: 3_1_1,
        name: 'Grandchild 3_1_1'
      }]
    },
    {
      id: 3_2,
      name: 'Child 3_2',
      children: [{
        id: 3_2_1,
        name: 'Grandchild 3_2_1'
      }]
    }]
  }]
}
```

#### TraversalTree JSON Output Generator
The two `TraversalTree` JSON output generators `DumpToJsonTopDown` and `DumpToJsonBottomUp` are pretty trivial as well, since all the work is done by the individual tree nodes:
```csharp
///
/// Dump the top-down traversal graph into JSON.
/// In this case, each parent node is populated
/// with a full hierarchical graph of all its
/// children, cf. https://www.jstree.com/docs/json.
/// summary>
public string DumpToJsonTopDown()
{
return m_startingElementNode
.DumpToJsonTopDown();
}
///
/// Dump the bottom-up traversal graph into JSON.
/// In this case, each child node is equipped with
/// a 'parent' pointer, cf.
/// https://www.jstree.com/docs/json/
/// summary>
public string DumpToJsonBottomUp()
{
List a = new List();
m_startingElementNode.DumpToJsonBottomUp( a, "#" );
return "[" + string.Join( ",", a ) + "]";
}
```
#### TreeNode JSON Output Generator
The two `TreeNode` JSON output generators are only slightly more complicated.
Here are the two formatting strings that they use:
```csharp
///
/// Format a tree node to JSON storing parent id
/// in child node for bottom-up structure.
/// summary>
const string _json_format_to_store_parent_in_child
= "{{"
+ "\"id\" : {0}, "
+ "\"name\" : \"{1}\", "
+ "\"parent\" : {2}}}";
///
/// Format a tree node to JSON storing a
/// hierarchical tree of children ids in parent
/// for top-down structure.
/// summary>
const string _json_format_to_store_children_in_parent
= "{{"
+ "\"id\" : {0}, "
+ "\"name\" : \"{1}\", "
+ "\"children\" : [{2}]}}";
```
Here are the two recursive functions implementing the JSON output:
```csharp
static string GetName( Element e )
{
return e.Name.Replace( "\"", "'" );
}
static string GetId( Element e )
{
return Options.StoreUniqueId
? "\"" + e.UniqueId + "\""
: e.Id.IntegerValue.ToString();
}
///
/// Add JSON strings representing all children
/// of this node to the given collection.
/// summary>
public void DumpToJsonBottomUp(
List json_collector,
string parent_id )
{
Element e = GetElementById( m_Id );
string id = GetId( e );
string json = string.Format(
_json_format_to_store_parent_in_child,
id, GetName( e ), parent_id );
json_collector.Add( json );
foreach( TreeNode node in m_childNodes )
{
node.DumpToJsonBottomUp( json_collector, id );
}
}
///
/// Return a JSON string representing this node and
/// including the recursive hierarchical graph of
/// all its all children.
/// summary>
public string DumpToJsonTopDown()
{
Element e = GetElementById( m_Id );
List json_collector = new List();
foreach( TreeNode child in m_childNodes )
{
json_collector.Add( child.DumpToJsonTopDown() );
}
string json_kids = string.Join( ",", json_collector );
string json = string.Format(
_json_format_to_store_children_in_parent,
GetId( e ), GetName( e ), json_kids );
return json;
}
```
#### Download and To Do
The current state of this project is available from
the [TraverseAllSystems GitHub repository](https://github.com/jeremytammik/TraverseAllSystems), and the version discussed above
is [release 2017.0.0.9](https://github.com/jeremytammik/TraverseAllSystems/releases/tag/2017.0.0.9).
The next step will consist of the Forge viewer extension implementation displaying a custom panel in the user interface hosting a tree view of the MEP system graphs and implementing two-way linking and selection functionality back and forth between the tree view nodes and the 2D and 3D viewer elements.
#### RevitLookup Updates
A couple of enhancement have been added to [RevitLookup](https://github.com/jeremytammik/RevitLookup) since
I last mentioned it, most lately
by [awmcc90](https://github.com/awmcc90)
and [Shayne Hamel](https://github.com/Shayneham)
to handle exceptions snooping MEP elements, electrical circuits, flex ducts and flex pipes.
Here are the diffs:
- [2017.0.0.5](https://github.com/jeremytammik/RevitLookup/compare/2017.0.0.4...2017.0.0.5) –
merged pull request #14 by Shayneham to handle exceptions snooping flex pipe and duct lacking levels etc.
- [2017.0.0.4](https://github.com/jeremytammik/RevitLookup/compare/2017.0.0.3...2017.0.0.4) –
merged pull request #13 by awmcc90 to skip mepSys.Elements for OST_ElectricalInternalCircuits category
Thank you very much for those improvements!
If you run into any issues with RevitLookup yourself, please fork the repository, implement and test your changes, and issue a pull request for me to integrate them back into the master branch.
Thank you!