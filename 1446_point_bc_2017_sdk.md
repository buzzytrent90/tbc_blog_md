---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.9
content_type: qa
optimization_date: '2025-12-11T11:44:16.041368'
original_url: https://thebuildingcoder.typepad.com/blog/1446_point_bc_2017_sdk.html
post_number: '1446'
reading_time_minutes: 4
series: geometry
slug: point_bc_2017_sdk
source_file: 1446_point_bc_2017_sdk.md
tags:
- elements
- filtering
- geometry
- references
- revit-api
- sheets
- transactions
title: The Building Coder
word_count: 733
---

### Point Boundary Condition and Revit 2017 SDK
I arrived safe and sound in San Francisco via Vancouver and am now working on the final preparations for
the [Forge DevCon](http://forge.autodesk.com/conference)
and [3D Web Fest](http://www.3dwebfest.com).
![Golden Gate Bridge and Marina](/p/2016/2016-06-13_greens/471_golden_gate_and_marina.jpg)
Before getting to that, here are some quick notes from my short exploration last week to produce a rather overdue list of the new Revit 2017 SDK samples and on creating a point boundary condition on a structural column:
- [New Revit 2017 SDK Samples](#2)
- [Creating Point Boundary Condition on End of Structural Column](#3)
#### New Revit 2017 SDK Samples
I used to pay much more attention in the past to the topic of new samples and Revit API functionality.
Obviously, the larger the API grows, the less difference each individual enhancement makes, and the more specialised the modifications become.
I trust you already discovered these samples yourself if they are important to you:
- Samples/CapitalizeAllTextNotes
- Samples/GenericStructuralConnection
- Samples/GeometryAPI/BRepBuilderExample
- Samples/PlacementOptions
- Structural Analysis SDK/Examples/CodeCheckingConcreteExample and CalculationPointsSelector
Here is the procedure I used to produce that list:

```
/a/lib/revit/2016/SDK $ find . -type d > /a/lib/revit/jeremy/ls2016.txt
/a/lib/revit/2016/SDK $ cd ../../2017/SDK/
/a/lib/revit/2017/SDK $ find . -type d > /a/lib/revit/jeremy/ls2017.txt
/a/lib/revit/2017/SDK $ cd ../../jeremy/
/a/lib/revit/jeremy $ diff ls2016.txt ls2017.txt
123c123
< ./REX SDK/Visual Studio templates/Items/Autodesk/Revit Extensions 2016
---
> ./REX SDK/Visual Studio templates/Items/Autodesk/Revit Extensions 2017
126c126
< ./REX SDK/Visual Studio templates/Projects/Autodesk/Revit Extensions 2016
---
> ./REX SDK/Visual Studio templates/Projects/Autodesk/Revit Extensions 2017
170a171,173
> ./Samples/CapitalizeAllTextNotes
> ./Samples/CapitalizeAllTextNotes/CS
> ./Samples/CapitalizeAllTextNotes/CS/Properties
384a388,390
> ./Samples/GenericStructuralConnection
> ./Samples/GenericStructuralConnection/CS
> ./Samples/GenericStructuralConnection/CS/Properties
385a392,395
> ./Samples/GeometryAPI/BRepBuilderExample
> ./Samples/GeometryAPI/BRepBuilderExample/CS
> ./Samples/GeometryAPI/BRepBuilderExample/CS/Properties
> ./Samples/GeometryAPI/BRepBuilderExample/CS/Resources
531a542,544
> ./Samples/PlacementOptions
> ./Samples/PlacementOptions/CS
> ./Samples/PlacementOptions/CS/Properties
695a709,710
> ./Structural Analysis SDK/Examples/ASCE-7-10/bin
> ./Structural Analysis SDK/Examples/ASCE-7-10/bin/Release
702a718,719
> ./Structural Analysis SDK/Examples/Concrete/CodeCheckingConcreteExample/bin
> ./Structural Analysis SDK/Examples/Concrete/CodeCheckingConcreteExample/bin/Release
709a727
> ./Structural Analysis SDK/Examples/Concrete/CodeCheckingConcreteExample/UIComponents/CalculationPointsSelector
714a733,734
> ./Structural Analysis SDK/Examples/Concrete/ConcreteCalculationsExample/bin
> ./Structural Analysis SDK/Examples/Concrete/ConcreteCalculationsExample/bin/Release
716a737,738
> ./Structural Analysis SDK/Examples/ExtensibleStorageDocumentation/bin
> ./Structural Analysis SDK/Examples/ExtensibleStorageDocumentation/bin/Release
719a742,743
> ./Structural Analysis SDK/Examples/ExtensibleStorageUI/bin
> ./Structural Analysis SDK/Examples/ExtensibleStorageUI/bin/Release
723a748,749
> ./Structural Analysis SDK/Examples/ResultsInRevit/QueryingResults/bin
> ./Structural Analysis SDK/Examples/ResultsInRevit/QueryingResults/bin/Release
725a752,753
> ./Structural Analysis SDK/Examples/ResultsInRevit/StoringResults/bin
> ./Structural Analysis SDK/Examples/ResultsInRevit/StoringResults/bin/Release
727a756,757
> ./Structural Analysis SDK/Examples/SectionPropertiesExplorer/bin
> ./Structural Analysis SDK/Examples/SectionPropertiesExplorer/bin/Release
```

#### Creating Point Boundary Condition on End of Structural Column
\*\*Question:\*\* I am trying to create a fixed boundary conditions at the beginning of a column following the example code in the Revit API help file RevitAPI.chm, but the endpoint reference of the `Curve` associated to the column analytical model keeps returning a null value.
Am I missing something?
\*\*Answer:\*\* Creating a boundary condition on end of column should work correctly.
Here is a code snippet from one of the Revit automatic regression tests.
It creates a point boundary condition on the ends of each analytical column:
```csharp
///
/// Create a point load on all
/// analytical column end points.
/// summary>
void CreatePointLoadOnColumnEnd( Document doc )
{
// Find all AM column instances in the document
FilteredElementCollector columns
= new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_ColumnAnalytical )
.WhereElementIsNotElementType();
foreach( AnalyticalModel am in columns )
{
Curve curve = am.GetCurve();
AnalyticalModelSelector selector
= new AnalyticalModelSelector( curve );
selector.CurveSelector
= AnalyticalCurveSelector.EndPoint;
Reference endPointRef
= am.GetReference( selector );
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "NewPointBoundaryConditions" );
BoundaryConditions newPointBC
= doc.Create.NewPointBoundaryConditions(
endPointRef,
TranslationRotationValue.Fixed, 0,
TranslationRotationValue.Spring, 1.0,
TranslationRotationValue.Fixed, 0,
TranslationRotationValue.Fixed, 0,
TranslationRotationValue.Fixed, 0,
TranslationRotationValue.Fixed, 0 );
newPointBC.SetOrientTo(
BoundaryConditionsOrientTo
.HostLocalCoordinateSystem );
tx.Commit();
}
}
}
```
You can run this code in the Revit Macro Manager to verify.
I added it
to [The Building Coder Samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2017.0.127.5](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2017.0.127.5)
in the module
[CmdNewLineLoad.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdNewLineLoad.cs#L30-L77).