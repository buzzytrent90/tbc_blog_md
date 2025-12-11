---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 3.4
content_type: qa
optimization_date: '2025-12-11T11:44:16.045731'
original_url: https://thebuildingcoder.typepad.com/blog/1449_traverse_mep_system.html
post_number: '1449'
reading_time_minutes: 6
series: mep
slug: traverse_mep_system
source_file: 1449_traverse_mep_system.md
tags:
- csharp
- elements
- family
- filtering
- parameters
- revit-api
- selection
- sheets
- transactions
- views
- mep
title: The Building Coder
word_count: 1292
---

### Traversing and Exporting all MEP System Graphs
The [Forge DevCon](http://forge.autodesk.com/conference) last week completed successfully.
I had a [full body 3D scan](https://www.artec3d.com/events/autodesk-forge-devcon-2016) created there in
the [Shapify Booth](https://www.artec3d.com/hardware/shapifybooth) and explained
on [The 3D Web Coder](http://thebuildingcoder.typepad.com/) how I
used [`sed`](https://en.wikipedia.org/wiki/Sed)
to [flip the axes of the resulting OBJ model](http://the3dwebcoder.typepad.com/blog/2016/06/flipping-obj-axes-with-texture-for-forge-viewer.html).
This week, I am sitting in the Autodesk offices at One Market in San Francisco, supporting the fourth [Cloud Accelerator](http://autodeskcloudaccelerator.com) taking place here.
One of the projects we are working on is from USC,
the [University of Southern California](http://www.usc.edu)
[Facilities Management](http://facilities.usc.edu)
[CAD Services](http://facilities.usc.edu/multisidebar_sublinks.asp?ItemID=236).
One of its goals is to interact with Revit MEP systems in the Forge viewer.
That requires traversing the MEP systems in the Revit model to store, recreate and represent their graph structures in the viewer.
Below, I present the Revit add-in that I have started implementing to generate and supply that information:
- [Revit MEP System Traversal](#2)
- [TraverseAllSystems Revit Add-in](#3)
- [Download](#4)
- [To do](#5)
- [Thanks to Mustafa Salaheldin](#6)
Before getting to that, here are a couple of pictures from the past weekend.
Saturday, I crossed the Golden Gate bridge and the Marin headlands to Sausalito:
[![Golden Gate Sausalito Walk](https://c2.staticflickr.com/8/7299/27152336233_188c438306_n.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157667256410223 "Golden Gate Sausalito Walk")
Sunday, I participated in
the [ecstatic dance event](http://ecstaticdance.org/SF)
([FB](https://www.facebook.com/groups/EcstaticSF)) in their great new location and enjoyed the views from
the [Buena Vista](http://sfrecpark.org/destination/buena-vista-park)
and [Presidio](https://en.wikipedia.org/wiki/Presidio_Park) parks:
[![Buena Vista and Presidio](https://c7.staticflickr.com/8/7690/27508672390_cd57261689_n.jpg)](https://www.flickr.com/photos/jeremytammik/albums/72157669044915670 "Buena Vista and Presidio")
#### Revit MEP System Traversal
Today, I present a simple Revit add-in that analyses all MEP systems in the model and determines the graph structure representing the connections between the systems elements.
I discussed a [simple MEP system traversal](http://thebuildingcoder.typepad.com/blog/2013/02/simple-mep-system-traversal.html) in 2013.
The graph information required in this case, however, requires the more advanced traversal algorithms determining the correct order of the individual system elements in the direction of the flow implemented by
the [TraverseSystem SDK sample](http://thebuildingcoder.typepad.com/blog/2009/06/revit-mep-api.html)
([2010](http://thebuildingcoder.typepad.com/blog/2009/09/the-revit-mep-api.html#6),
[2011](http://thebuildingcoder.typepad.com/blog/2010/05/the-revit-mep-2011-api.html#samples)) for mechanical systems and
the [AdnRme sample](http://thebuildingcoder.typepad.com/blog/2012/05/the-adn-mep-sample-adnrme-for-revit-mep-2013.html)
([GitHub repo](https://github.com/jeremytammik/AdnRme)) for electrical ones.
#### TraverseAllSystems Revit Add-in
I implemented a simple C# .NET Revit API add-in that performs the following steps:
- Process the BIM in read-only mode.
- Retrieve all MEP systems from the model.
- Filter out the ones we are interested in by applying the `IsDesirableSystemPredicate` method.
- For each of these systems, instantiate a `TraversalTree` object, scavenged from the Revit SDK TraverseSystem sample.
- Create a temporary output folder to store the resulting XML files storing the graph information.
- Export a separate XML file for each system.
Here is the main implementation file of the external command:
```csharp
[Transaction( TransactionMode.ReadOnly )]
public class Command : IExternalCommand
{
///
/// Return true to include this system in the
/// exported system graphs.
/// summary>
static bool IsDesirableSystemPredicate( MEPSystem s )
{
return s is MechanicalSystem || s is PipingSystem
&& !s.Name.Equals( "unassigned" )
&& 1 < s.Elements.Size;
}
///
/// Create a and return the path of a random temporary directory.
/// summary>
static string GetTemporaryDirectory()
{
string tempDirectory = Path.Combine(
Path.GetTempPath(), Path.GetRandomFileName() );
Directory.CreateDirectory( tempDirectory );
return tempDirectory;
}
public Result Execute(
ExternalCommandData commandData,
ref string message,
ElementSet elements )
{
UIApplication uiapp = commandData.Application;
UIDocument uidoc = uiapp.ActiveUIDocument;
Application app = uiapp.Application;
Document doc = uidoc.Document;
FilteredElementCollector allSystems
= new FilteredElementCollector( doc )
.OfClass( typeof( MEPSystem ) );
int nAllSystems = allSystems.Count();
IEnumerable desirableSystems
= allSystems.Cast().Where(
s => IsDesirableSystemPredicate( s ) );
int nDesirableSystems = desirableSystems
.Count();
string outputFolder = GetTemporaryDirectory();
int n = 0;
foreach( MEPSystem system in desirableSystems )
{
Debug.Print( system.Name );
FamilyInstance root = system.BaseEquipment;
// Traverse the system and dump the
// traversal graph into an XML file
TraversalTree tree = new TraversalTree( system );
if( tree.Traverse() )
{
string filename = system.Id.IntegerValue.ToString();
filename = Path.ChangeExtension(
Path.Combine( outputFolder, filename ), "xml" );
tree.DumpIntoXML( filename );
// Uncomment to preview the
// resulting XML structure
//Process.Start( fileName );
++n;
}
}
string main = string.Format(
"{0} XML files generated in {1} ({2} total"
+ "systems, {3} desirable):",
n, outputFolder, nAllSystems,
nDesirableSystems );
List system_list = desirableSystems
.Select( e =>
string.Format( "{0}({1})", e.Id, e.Name ) )
.ToList();
system_list.Sort();
string detail = string.Join( ", ",
system_list.ToArray() );
TaskDialog dlg = new TaskDialog( n.ToString()
+ " Systems" );
dlg.MainInstruction = main;
dlg.MainContent = detail;
dlg.Show();
return Result.Succeeded;
}
}
```
I ran the command in the RME advanced sample project included with the standard Revit installation, `rme_advanced_sample_project.rvt`.
The result of doing so looks like this:
![TraverseAllSystems result in rme_advanced_sample_project.rvt](img/traverse_all_systems_in_rme_advanced_sample_project.png)
It contains 240 MEP systems, 51 of which were deemed 'desirable' by the `IsDesirableSystemPredicate` method, of which only 35 produced any interesting graph data, exported to individual XML files in a random temporary directory.
#### Download
The current state of this project is available from
the [TraverseAllSystems GitHub repository](https://github.com/jeremytammik/TraverseAllSystems), and the version discussed above
is [release 2017.0.0.1](https://github.com/jeremytammik/TraverseAllSystems/releases/tag/2017.0.0.1).
#### To Do
The next step will be to implement a Forge viewer extension displaying a custom panel in the user interface hosting a tree view of the MEP system graphs and implementing two-way linking and selection functionality back and forth between the tree view nodes and the 2D and 3D viewer elements.
We also need to figgure out how to transport the graph information from the Revit add-in to the Forge viewer.
Presumably, we will encode it in JSON instead of XML, to start with, to make it easier to handle directly in JavaScript, e.g. by implementing a viewer extension making use of [jstree](https://www.jstree.com/docs/json) to interact with the graph.
Here are some of the storage options:
- Store the graph data in a stand-alone cloud-based repository and link it with the viewer elements dynamically
- Store the graph data as neighbourship relationships in each MEP system element, for instance in a shared parameter.
- Store the entire graph data in one single JSON structure, for instance on each MEP system base equipment element.
These options can obviously be combined, and even all implemented at once.
Probaly, the easiest way to transport the data from the BIM to the Forge platform will be to store it in shared parameter data on Revit elements.
Then it will be automatically included and handled by the standard Forge translation process for Revit RVT files.
#### Thanks to Mustafa Salaheldin
One last important point before closing.
In the past weeks,
[Mustafa Salaheldin](http://forums.autodesk.com/t5/user/viewprofilepage/user-id/1227311) has answered more cases on
the [Revit API forum](http://forums.autodesk.com/t5/revit-api/bd-p/160) than any other person before him ever was able to do in the past:
![Mustafa Salaheldin ranking](img/mustafa_salaheldin_ranking_2.png)
Nobody ever reached that ranking before.
I cannot even imagine how he does it.
Thank you very much, Mustafa!
It is extremely appreciated by the entire community!