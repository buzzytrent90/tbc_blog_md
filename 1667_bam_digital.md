---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 2.8
content_type: qa
optimization_date: '2025-12-11T11:44:16.494131'
original_url: https://thebuildingcoder.typepad.com/blog/1667_bam_digital.html
post_number: '1667'
reading_time_minutes: 3
series: general
slug: bam_digital
source_file: 1667_bam_digital.md
tags:
- geometry
- parameters
- revit-api
- sheets
- transactions
- views
title: Bam Digital
word_count: 553
---

### Digital Construction and Drawing a Model Line
I am attending the BAM *Digital Construction Live* event in the UK and presenting on Forge for that domain.
You can see what's going on here looking for
the [#bamdigital](https://twitter.com/search?q=%23bamdigital) hashtag.
On the way here, I visited my brother and passed by the interesting climbing areas
at [Cheddar Gorge](https://en.wikipedia.org/wiki/Cheddar_Gorge)
and [Symonds Yat](https://en.wikipedia.org/wiki/Symonds_Yat):
![Cheddar Gorge](img/119_jeremy_leading_600.jpg)
In the latter, we climbed the Long Rock Pinnacle via Whitt:
![Long Rock Pinnacle](img/143_jeremy_long_rock_pinnacle_800x600.jpg)
Today, I'll share my slide deck from this event and welcome my colleague Xiaodong answering his first Revit API cases:
- [Forge for Digital Construction](#2)
- [Welcome Xiaodong and invoking the \*Draw Model Line\* command](#3)
#### Forge for Digital Construction
As said, I am attending this event with my Autodesk colleagues Az Jasat and Manu Venugopal, presenting on Insight and Connection.
Insight is covered by Manu, discussing Project IQ and machine learning enhanced analytics for automated risk assessment.
I am presenting on Forge for the digital construction process and connecting to the BIM360 products.
I already explained the main concepts from my point of view in
the [overview of Forge for AEC and BIM360](http://thebuildingcoder.typepad.com/blog/2018/06/forge-for-aec-and-bim360-overview.html) and
the [BIM360 and Forge for AEC message and samples](http://thebuildingcoder.typepad.com/blog/2018/06/bim360-and-forge-for-aec-real-message-and-samples.html).
Here is the final slide deck summarising those points in
the [BAM Forge for Digital Construction slides](zip/bam_bim360_forge_aec_slides.pdf).
![Manu and Az at BAM Digital Construction Live](img/187_manu_and_az_722x380.jpg)
#### Welcome Xiaodong and Invoking the \*Draw Model Line\* Command
Many congratulations to my colleague Xiaodong Liang for diving into the Revit API and starting to answer cases!
Here is one of his first:
\*\*Question:\*\* How can I programmatically invoke the \*Draw Model Line\* command?
\*\*Answer:\*\* If your workflow is to simply to invoke the built-in \*Draw Model Line\* command as is, you could find the command id and execute it using `PostCommand`:
```csharp
Autodesk.Revit.UI.RevitCommandId cmd_id
= RevitCommandId.LookupPostableCommandId(
PostableCommand.ModelLine );
uiapp.PostCommand( cmd_id );
```
If your workflow is to create a model line yourself with the parameters the user inputs, you can use the following:
```csharp
UIApplication uiApp = commandData.Application;
Application rvtApp = uiApp.Application;
UIDocument uiDoc = uiApp.ActiveUIDocument;
Document doc = uiDoc.Document;
using( Transaction transaction = new Transaction( doc ) )
{
transaction.Start( "Create Model Line By Me" );
XYZ startPoint = XYZ.Zero;
XYZ endPoint = new XYZ( 10, 10, 0 );
Line geomLine = Line.CreateBound( startPoint, endPoint );
// Create a geometry plane in Revit application memory
XYZ origin = XYZ.Zero;
XYZ normal = XYZ.BasisZ;
Plane geomPlane = Plane.CreateByNormalAndOrigin(
normal, origin );
// Create a sketch plane in current document
SketchPlane sketch = SketchPlane.Create(
doc, geomPlane );
ModelLine modelLine = doc.Create.NewModelCurve(
geomLine, sketch ) as ModelLine;
transaction.Commit();
}
```
Xiaodong adds:
> Though it is a simple question, it took me some time to test out the working codes, and I learned some valuable knowledge of Revit API.
> Stumblingly at the starting point of the journey into the Revit API :-)
Many thanks to Xiaodong for the nice and comprehensive answer!
It looks like a great start!
I wish you much success and lots of fun going further.