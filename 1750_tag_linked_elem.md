---
post_number: "1750"
title: "Tag Linked Elem"
slug: "tag_linked_elem"
author: "Jeremy Tammik"
tags: ['doors', 'elements', 'family', 'filtering', 'geometry', 'levels', 'references', 'revit-api', 'selection', 'sheets', 'transactions', 'views', 'walls']
source_file: "1750_tag_linked_elem.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1750_tag_linked_elem.html"
---

### Tagging a Linked Element
The linked file enhancements introduced in the Revit 2014 API obviously need more awareness:
- [Link enhancements – conversion of geometric references](#2)
- [Tagging a linked element](#3)
- [Using the stable representation to tag a linked element](#4)
- [List all untagged doors](#5)
![Tag linked element](img/tag_linked_element.jpg)
#### Link Enhancements – Conversion of Geometric References
`CreateLinkReference` was introduced way back in
the [Revit 2014 API](https://thebuildingcoder.typepad.com/blog/2013/04/whats-new-in-the-revit-2014-api.html).
Conversion of geometric references in links is mentioned as one of the major enhancements:
The API calls:
- [Reference.LinkedElementId](http://www.revitapidocs.com/2020/97813744-6e64-00a7-da5c-b2c6de7919ad.htm) – The id of the top-level element in the linked document that is referred to by this reference.
- [Reference.CreateLinkReference(RevitLinkInstance)](http://www.revitapidocs.com/2020/919d7d3f-f8c2-eb12-4069-0022c20fa13a.htm) – Creates a `Reference` from a `Reference` in an RVT Link.
- [Reference.CreateReferenceInLink()](http://www.revitapidocs.com/2020/20a8bee7-2378-c0a6-36f0-07ca42eaedc3.htm) – Creates a `Reference` in an RVT Link from a `Reference` in the RVT host file.
allow conversion between `Reference` objects which reference only the contents of the link and `Reference` objects which reference the host.
This allows an application, for example, to look at the geometry in the link, find the needed face, and convert the reference to that face into a reference in the host suitable for use to place a face-based instance.
Also, they allow you to obtain a reference in the host (e.g., from a dimension or family) and convert it to a reference in the link, suitable for use in `Element.GetGeometryObjectFromReference`.
This enhancement was often overlooked, and several questions were raised on how to tag an element in a linked file.
#### Tagging a Linked Element
Ilia Ivanov used these methods to answer his own question in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [tagging linked elements using Revit API](https://forums.autodesk.com/t5/revit-api-forum/tagging-linked-elements-using-revit-api/m-p/8669001):
\*\*Question:\*\* Hello, Is it possibly to tag a linked element?
And also retrieve the reference of the tagged linked element?
\*\*Answer:\*\* Hello, I have done it:
```csharp
RevitLinkInstance link = doc.GetElement(
tag.TaggedElementId.LinkInstanceId )
as RevitLinkInstance;
Reference refer = new Reference(
link.GetLinkDocument()
.GetElement( tag.TaggedElementId.LinkedElementId ) )
.CreateLinkReference( link );
```
For a bit more context, here is a slightly nonsensical sample method to tag all walls in all linked documents, placing all tags in one single constant spot:
```csharp
///
/// Tag all walls in all linked documents
/// summary>
void TagAllLinkedWalls( Document doc )
{
// Point near my wall
XYZ xyz = new XYZ( -20, 20, 0 );
// At first need to find our links
FilteredElementCollector collector
= new FilteredElementCollector( doc )
.OfClass( typeof( RevitLinkInstance ) );
foreach( Element elem in collector )
{
// Get linkInstance
RevitLinkInstance instance = elem
as RevitLinkInstance;
// Get linkDocument
Document linkDoc = instance.GetLinkDocument();
// Get linkType
RevitLinkType type = doc.GetElement(
instance.GetTypeId() ) as RevitLinkType;
// Check if link is loaded
if( RevitLinkType.IsLoaded( doc, type.Id ) )
{
// Find walls for tagging
FilteredElementCollector walls
= new FilteredElementCollector( linkDoc )
.OfCategory( BuiltInCategory.OST_Walls )
.OfClass( typeof( Wall ) );
// Create reference
foreach( Wall wall in walls )
{
Reference newRef = new Reference( wall )
.CreateLinkReference( instance );
// Create transaction
using( Transaction tx = new Transaction( doc ) )
{
tx.Start( "Create tags" );
IndependentTag newTag = IndependentTag.Create(
doc, doc.ActiveView.Id, newRef, true,
TagMode.TM_ADDBY_MATERIAL,
TagOrientation.Horizontal, xyz );
// Use TaggedElementId.LinkInstanceId and
// TaggedElementId.LinkInstanceId to retrieve
// the id of the tagged link and element:
LinkElementId linkId = newTag.TaggedElementId;
ElementId linkInsancetId = linkId.LinkInstanceId;
ElementId linkedElementId = linkId.LinkedElementId;
tx.Commit();
}
}
}
}
}
```
Many thanks to Ilia for sharing this!
#### Using the Stable Representation to Tag a Linked Element
In another extensive thread
on [highlighting and tagging linked elements](http://forums.autodesk.com/t5/revit-api/highlight-and-tag-linked-elements/m-p/5294217).
Carolina Machado suggested an alternative approach and less official solution to tag a linked element using the `ParseFromStableRepresentation` method instead:
> Using RevitLookup and a post from your blog, I noticed that the Stable Representation of references in linked instances conform to the following pattern:
>  `revitLinkInstance.UniqueId

 +":0:RVTLINK/" + revitLinkType.UniqueId

 +":" + element.Id.ToString()`
> Using this string, it is possible to get the `Reference` through `Reference.ParseFromStableRepresentation` method and then use it to tag the element.
Many thanks to Carolina for sharing this!
#### List All Untagged Doors
On a another tagging topic, however with no links involved, here are two suggestions by my colleague Naveen Kumar and
Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) Ignatovich, aka Александр Игнатович,
answering a whole slew of questions on how to retrieve all untagged doors in the model:
- [I want to check whether tag is present on door by API](https://forums.autodesk.com/t5/revit-api-forum/i-want-to-check-whether-tag-is-present-on-door-by-api-how-should/td-p/8532032)
- [How to get relation of element with its tag or its label](https://forums.autodesk.com/t5/revit-api-forum/how-to-gets-relation-of-element-with-its-tag-or-its-label/td-p/8602124)
- [How to verify label on element using Revit API](https://forums.autodesk.com/t5/revit-api-forum/how-to-verify-label-on-element-using-revit-api/td-p/8594801)
\*\*Question:\*\* Can we determine the relationship between a tag and its tagged element?
I can retrieve all independent tags of particular category.
E.g., having 6 doors, I can retrieve the 6 door tags.
Suppose one of doors does not have tag.
How can I find the particular door lacking a tag?
In other words, how to find relation between element category and element tag category.
\*\*Answer:\*\* Try using the below code. It will highlight the elements that are not tagged:
```csharp
FilteredElementCollector doors
= new FilteredElementCollector( doc )
.OfCategory( BuiltInCategory.OST_Doors )
.OfClass( typeof( FamilyInstance ) );
IEnumerable tags
= new FilteredElementCollector( doc )
.OfClass( typeof( IndependentTag ) )
.Cast();
IList untagged_elements
= new List();
foreach( Element e in doors )
{
if( !tags.Any( q
=> q.TaggedLocalElementId == e.Id ) )
{
untagged_elements.Add( e.Id );
}
}
uidoc.Selection.SetElementIds( untagged_elements );
uidoc.RefreshActiveView();
```
\*\*Answer 2:\*\* To check whether a specific door is untagged, you can find all `IndependentTag` elements present in the document of `OST_DoorTags` category and the door elements that they are tagging like this:
```csharp
var collector = new FilteredElementCollector( doc )
.OfClass( typeof( IndependentTag ) )
.OfCategory( BuiltInCategory.OST_DoorTags );
var doorTagsIds
= new HashSet(
collector.OfType()
.Select( x => x.GetTaggedLocalElement()?.Id )
.Where( x => x != null ) );
```
Then you can iterate over your doors collection and check if `doorTagsIds` contains the `door.Id`.
Both of these suggestions can probably be speeded up by storing the entire relationship between the tags and the tagged elements in a dictionary and inverting that relationship, as I explained in the recent thread
on [`FilteredElementCollector` – unreferenced sections only](https://forums.autodesk.com/t5/revit-api-forum/filteredelementcollector-unreferenced-sections-only/m-p/8773472).