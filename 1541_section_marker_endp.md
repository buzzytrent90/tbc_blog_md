---
post_number: "1541"
title: "Section Marker Endp"
slug: "section_marker_endp"
author: "Jeremy Tammik"
tags: ['elements', 'family', 'filtering', 'parameters', 'references', 'revit-api', 'sheets', 'transactions', 'views']
source_file: "1541_section_marker_endp.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1541_section_marker_endp.html"
---

### TTT to Obtain Section Marker Endpoint
We discussed several examples of using the temporary transaction trick TTT in the past, and it is also mentioned in The Building Coder topic group
on [handling transactions and transaction groups](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.53).
Here is a new exquisitely subtle variant for you to enjoy, provided by
Frank [@Fair59](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/2083518) Aarssen.
He uses it to answer
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) question
on [how to get the coordinates of the endpoints for a section marker line segment](https://forums.autodesk.com/t5/revit-api-forum/how-can-i-get-the-coordinates-of-the-endpoints-for-a-section/m-p/6928342),
an issue that cannot be solved, according to the Revit development team:
\*\*Question:\*\* I have a section marker that I would like to rotate around one of the endpoints of the line segment leader, but I haven't been able to figure out how to determine the endpoint coordinates:
![Section marker coordinates](img/section_marker_coordinates.png)
I'm able to get the bounding box of the overall section marker but not these specific coordinates.
How can I do this?
My bigger problem involves reversing the effects of the built-in "Mirror Project" tool, and part of that includes reference section markers inside of another section that has been flipped.
I would like to fix their orientation by rotating 180 degrees around the endpoint.
\*\*Answer:\*\* The development team says that the section marker element is not exposed as a specific element type, so this position cannot be read at this time.
However, Frank provided a workaround.
A transaction needs to be active when you call the method, as the method temporarily hides the section tag and viewer_reference_label_text:
```csharp
Line GetSectionLine(
Element section,
View view )
{
const double correction = 21.130014403 / 304.8;
Document doc = section.Document;
Category cat = section.Category;
if( null == cat )
{
throw new ArgumentException(
"Section has null category" );
}
if( BuiltInCategory.OST_Viewers
!= (BuiltInCategory) (cat.Id.IntegerValue) )
{
throw new ArgumentException(
"Expected section with OST_Viewers category" );
}
FilteredElementCollector views
= new FilteredElementCollector( doc )
.OfClass( typeof( View ) );
View viewFromSection = null;
foreach( View v in views )
{
if( section.Name == v.Name
&& section.GetTypeId() == v.GetTypeId() )
{
viewFromSection = v;
break;
}
}
if( viewFromSection == null ) return null;
ViewFamilyType vType = doc.GetElement(
section.GetTypeId() ) as ViewFamilyType;
BoundingBoxXYZ bb1 = null;
using( SubTransaction st1 = new SubTransaction( doc ) )
{
st1.Start();
Parameter par = vType.get_Parameter(
BuiltInParameter.SECTION_TAG );
par.Set( ElementId.InvalidElementId );
par = vType.get_Parameter(
BuiltInParameter.VIEWER_REFERENCE_LABEL_TEXT );
par.Set( string.Empty );
view.Scale = 1;
doc.Regenerate();
bb1 = section.get_BoundingBox( view );
st1.RollBack();
}
BoundingBoxXYZ bb = section.get_BoundingBox( view );
XYZ pt1 = bb.Min;
XYZ pt2 = bb.Max;
if( bb1 != null )
{
pt1 = bb1.Min;
pt2 = bb1.Max;
}
XYZ Origin = viewFromSection.Origin;
XYZ ViewBasisX = viewFromSection.RightDirection;
XYZ ViewBasisY = viewFromSection.ViewDirection;
if( ViewBasisX.X < 0 ^ ViewBasisX.Y < 0 )
{
double d = pt1.Y;
pt1 = new XYZ( pt1.X, pt2.Y, pt1.Z );
pt2 = new XYZ( pt2.X, d, pt2.Z );
}
XYZ ToPlane1 = pt1.Add( ViewBasisY.Multiply(
ViewBasisY.DotProduct( Origin.Subtract( pt1 ) ) ) );
XYZ ToPlane2 = pt2.Subtract( ViewBasisY.Multiply(
ViewBasisY.DotProduct( pt2.Subtract( Origin ) ) ) );
XYZ correctionVector = ToPlane2.Subtract( ToPlane1 )
.Normalize().Multiply( correction );
XYZ endPoint0 = ToPlane1.Add( correctionVector );
XYZ endPoint1 = ToPlane2.Subtract( correctionVector );
return Line.CreateBound( endPoint0, endPoint1 );
}
```
\*\*Response:\*\* I have a few questions:
1. Where did you come up with the `const double correction = 21.130014403 / 304.8` value?
2. How do you distinguish between section vs. callout markers since they are both of category `OST_Viewers`?
3. Would this work for section markers that are only references (such as ones that reference drafting views)?
4. How do you know which endpoint is associated with the tail and which one is associated with the head?
\*\*Answer 1:\*\* The bounding box of a section line is the result of a number of elements:
- the line
- the section head
- the reference label text
- the cycle symbol
![Section marker bounding box](img/section_marker_bounding_box.png)
In the method, I "hide" 2 and 3.
You can't hide the cycle symbol, so the result is too large.
I simply added the resulting line without the correction to the project and measured the surplus length.
After checking that it is always the same, I implemented the constant `correction = 21.130014403 / 304.8`.
The number 304.8 is the number of millimetres in a foot, so the division by that number converts from feet to mm:

```
  inch = 25.4
  foot = 12 * inch = 304.8
  length_in_mm = length_in_feet / foot
```

\*\*Answer 2:\*\* I didn't distinguish between section vs. callout markers, I merely presumed you had a section marker.
There are 2 ways to test:
```csharp
if( _vType.FamilyName == "Section" )
if( _viewFromSection.GetType()
== typeof( ViewSection ) )
```
Callouts are PlanViews or Elevations, as far as I know.
\*\*Answer 3:\*\* I don't know.
\*\*Answer 4:\*\* I'm afraid that is truly hidden. The Section Head and Tail stay in the same position when flipping the section, but the viewDirection and RightDirection changes.
The most you can get is a line in the View.RightDirection
```csharp
XYZ _direction = ToPlane2.Subtract( ToPlane1 ).
Normalize();
return _direction.IsAlmostEqualTo( ViewBasisX )
? Line.CreateBound( endPoint0, endPoint1 )
: Line.CreateBound( endPoint1, endPoint0 );
```
Many thanks to Frank for this tricky solution, and many other extremely helpful and in-depth answers in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) as well!