---
post_number: "1845"
title: "Raycast Linked"
slug: "raycast_linked"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'geometry', 'levels', 'references', 'revit-api', 'selection', 'sheets', 'views', 'walls']
source_file: "1845_raycast_linked.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1845_raycast_linked.html"
---

### Using ReferenceIntersector with a Linked File
Here are our topics for today:
- [What is Forge in 90 seconds](#2)
- [Locally opening RVT file managed by BIM360](#3)
- [Programming tools for Robobat](#5)
- [Using `ReferenceIntersector` in linked files](#5)
#### What is Forge in 90 Seconds
A new quick, high-level, non-technical overview of [Autodesk Forge](https://forge.autodesk.com) is
now available on YouTube, the 90-second [\*What is Forge\* introduction video](https://youtu.be/LvtwXf0AgME).
This short introduction showcases the endless possibilities and some innovative solutions and workflows built with it.
Featured footage includes demos from Moicon, CADshare, Xinaps, Project Frog, and InsiteVR.
#### Locally Opening RVT File Managed by BIM360
\*\*Question:\*\* How can I open an RVT file managed by BIM360 locally using the Revit API, e.g., using `OpenAndActivateDocument`?
\*\*Answer:\*\* What exactly do you mean by \*RVT file managed by BIM 360 locally\*?
Is the file stored in \*%localappdata%\autodesk\revit\CollboarationCache\*, coming from RCM?
\*\*Response:\*\* I am using BIM360 to manage the RVT file.
I would now like to add some local processing using full Revit plus my custom add-in.
Of course, I could use the [Forge Data Management API](https://forge.autodesk.com/api/data-management-cover-page/) to
download it, but I hope there is some way to achieve this directly using the Revit API.
\*\*Answer:\*\* Currently, Revit only supports opening RCM-based cloud models.
Revit doesn’t yet support opening files directly uploaded to BIM360.
\*\*Response:\*\* What if the file was uploaded to BIM360 via RCW?
Does the Revit API provide access to it then?
\*\*Answer:\*\* If the file was uploaded to BIM360 via RCW, officially termed as Initiate, and only accessible through the Revit UI and API, then yes, it can be opened via `OpenAndActivateDocument`.
Here is a documentation on [opening a BIM360 file in Design Automation](https://stackoverflow.com/questions/61098804/autodesk-forge-design-automation-error-opening-a-model-how-to-bypass-dialog/61101203#61101203); note that this file is an eTransmitted Workshared file.
If DA is not an option here, the right overload of to use in the Revit API is the `UIApplication` method `OpenAndActivateDocument` taking the arguments `ModelPath`, `OpenOptions`, `Boolean` and `IOpenFromCloudCallback`.
You will also have to call the `ModelPathUtils` method `ConvertCloudGUIDsToCloudPath` taking `String`, `Guid`, `Guid` first.
The two guids are the project id and model id used by RCM to identify the project and model.
For an example, please refer to the discussion
on [how to get project Guid and model Guid from `PathName`](https://stackoverflow.com/questions/51370445/how-to-get-project-guid-and-model-guid-from-pathname).
#### Programming Tools for Robobat
\*\*Question:\*\* I would like to program using the Robobat API.
The documentation in RSA 2020 suggests using Visual Studio 2008.
The current version of Visual Studio is 2020.
Can you confirm what version of Visual Studio I can use for programming the Robobat API?
\*\*Answer:\*\* The Robot SDK can be installed from any version the standard Robot Structural Analysis installer.
Select \*Tools and Utilities\* followed by \*Autodesk Robot Structural Analysis Professional SDK\*.
The SDK includes the document \*Getting Started Guide Robot API.pdf\* with information about Visual Studio.
The statement about VS 2008 should be more precise and should in fact suggest VS2008 and all later versions.
A few other things worth knowing:
- Starting from RSA 2012 all projects need to be compiled in x64 version
- You can use any programming language you like that supports .NET
- RSA 2020 uses the .NET Framework 4.7
- RSA 2021 uses the .NET Framework 4.8
#### Using ReferenceIntersector in Linked Files
Diving in a bit deeper into the Revit API,
[Ilia Ivanov](https://www.linkedin.com/in/ilya-ivanov-298997161/?locale=en_US) shares a nice example
of [using `ReferenceIntersector` in linked files](https://forums.autodesk.com/t5/revit-api-forum/using-referenceintersector-in-linked-files/m-p/9516302):
I faced the problem of how to use a `ReferenceIntersector` with `RevitLinkInstance` elements.
I achieved a solution using filters etc. that works rather well and I would like to share with the community.
My task was to add opening family instances to each wall intersected by a crossing pipe.
I decided to implement that using `IUpdater`.
My solution worked well with non-linked walls, using filters to make the `ReferenceIntersector` find only walls.
However, when I started preparing the solution for linked walls, I faced a problem: references from the list `ReferenceWithContext` contain the id of a `RevitLinkInstance` and not the id of the target wall, so I couldn't gather the linked walls.
I looked through the post
on [using `ReferenceIntersector` in linked files](https://thebuildingcoder.typepad.com/blog/2015/07/using-referenceintersector-in-linked-files.html) and
found that we can't get the references from a linked file.
Debugging my code further, I realized a solution.
The `Reference` class has a property `LinkedElementId`.
Given a `RevitLinkInstance` and the property of the element in the linked file we can retrieve the element in the linked file.
But, when my pipe crosses the linked wall, the `ReferenceWithContext` list contains some amount of elements with the same ids (LinkedElementId, Id), maybe because it also gathers geometry like faces etc.
To distinguish the values in this list, I had to create a custom `EqualityCompare`.
Finally, the code worked perfectly.
Below is the method to gather count of intersected walls.
Note that you need to run it in a 3D view without section boxes achieve good results.
```csharp
public void GetWalls( UIDocument uidoc )
{
Document doc = uidoc.Document;
Reference pipeRef = uidoc.Selection.PickObject(
ObjectType.Element );
Element pipeElem = doc.GetElement( pipeRef );
LocationCurve lc = pipeElem.Location as LocationCurve;
Curve curve = lc.Curve;
ReferenceComparer reference1 = new ReferenceComparer();
ElementFilter filter = new ElementCategoryFilter(
BuiltInCategory.OST_Walls );
FilteredElementCollector collector
= new FilteredElementCollector( doc );
Func isNotTemplate = v3 => !(v3.IsTemplate);
View3D view3D = collector
.OfClass( typeof( View3D ) )
.Cast()
.First( isNotTemplate );
ReferenceIntersector refIntersector
= new ReferenceIntersector(
filter, FindReferenceTarget.Element, view3D );
refIntersector.FindReferencesInRevitLinks = true;
IList referenceWithContext
= refIntersector.Find(
curve.GetEndPoint( 0 ),
(curve as Line).Direction );
IList references
= referenceWithContext
.Select( p => p.GetReference() )
.Distinct( reference1 )
.Where( p => p.GlobalPoint.DistanceTo(
curve.GetEndPoint( 0 ) ) < curve.Length )
.ToList();
IList walls = new List();
foreach( Reference reference in references )
{
RevitLinkInstance instance = doc.GetElement( reference )
as RevitLinkInstance;
Document linkDoc = instance.GetLinkDocument();
Element element = linkDoc.GetElement( reference.LinkedElementId );
walls.Add( element );
}
TaskDialog.Show( "Count of wall", walls.Count.ToString() );
}
public class ReferenceComparer : IEqualityComparer
{
public bool Equals( Reference x, Reference y )
{
if( x.ElementId == y.ElementId )
{
if( x.LinkedElementId == y.LinkedElementId )
{
return true;
}
return false;
}
return false;
}
public int GetHashCode( Reference obj )
{
int hashName = obj.ElementId.GetHashCode();
int hashId = obj.LinkedElementId.GetHashCode();
return hashId ^ hashId;
}
}
```
Unfortunately, no time to add comments to the code.
Hope it may help somebody.
Here is [reference_intersector_in_linked_files.zip](zip/reference_intersector_in_linked_files.zip) containing
a sample project with a macro to test.
It includes two Revit files:
- Architectural link.rvt
- CountOfLinkedWalls.rvt
The latter hosts the former as a linked file and contains a macro module named `CountOfIntersectedWalls` defining the method `GetWalls` listed above:
![ReferenceIntersector in linked files](img/reference_intersector_in_linked_files.png "ReferenceIntersector in linked files")
In case of need, here is also a code snippet to get a `StableRepresentation` for a linked wall's exterior face:
```csharp
public string GetFaceRefRepresentation(
Wall wall,
Document doc,
RevitLinkInstance instance )
{
Reference faceRef = HostObjectUtils.GetSideFaces(
wall, ShellLayerType.Exterior ).FirstOrDefault();
Reference stRef = faceRef.CreateLinkReference( instance );
string stable = stRef.ConvertToStableRepresentation( doc );
return stable;
}
```
Many thanks to Ilia for this nice solution and very helpful explanation!
Added to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2021.0.148.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2021.0.148.2).