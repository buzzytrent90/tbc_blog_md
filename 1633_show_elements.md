---
ai_optimized: true
author: Jeremy Tammik
complexity_score: 4.6
content_type: qa
optimization_date: '2025-12-11T11:44:16.430367'
original_url: https://thebuildingcoder.typepad.com/blog/1633_show_elements.html
post_number: '1633'
reading_time_minutes: 5
series: elements
slug: show_elements
source_file: 1633_show_elements.md
tags:
- elements
- family
- filtering
- parameters
- revit-api
- sheets
- transactions
- views
title: Show Elements
word_count: 1047
---

### Switch View or Document by Showing Elements
A recent discussion on using the `ShowElements` method to toggle between documents and views brought up a few interesting points:
- [Open and active an unsaved document](#2)
- [Zoom to selected elements](#3)
- [Toggle between documents and views](#4)
#### Open and Active an Unsaved Document
Normally, you can open and switch between documents using the `OpenAndActivateDocument` method.
However, it requires a file path, which may not have been defined yet, in the case of a newly created document.
In that case, you can open and switch between different documents and their views by calling the `UIDocument` `ShowElements` method instead.
This was discussed again in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [changing the active document](https://forums.autodesk.com/t5/revit-api-forum/change-active-document/m-p/7787792),
and previously
in [how to open and active a new document that is not saved](https://forums.autodesk.com/t5/revit-api-forum/how-to-open-and-active-a-new-document-that-is-not-saved/m-p/7710749):
\*\*Question:\*\* When you have 2 Revit projects open and want to switch between them, it can be done with:
```csharp
application.OpenAndActivateDocument(file);
```
I used `Document.PathName` to get the filename required.
Now I run into problems with files on Revit Server and BIM 360 docs, because their `Document.Pathname` stays empty.
So, my question is, how do I switch between those documents (when `Document.Pathname` is empty)?
Is there a way to switch between documents without having to specify the file name?
\*\*Answer:\*\* The only way I have found is indirectly via `UIDocument.ShowElements`.
You pick an element from the DB document you want to change to, create a `UIDocument` object from a DB document and use `UIDocument.ShowElements`. You have to handle the occasional “No good view found” dialogue, but it seems to always switch the active document regardless of what is found. Helps if it is a `View` specific element.
Not sure if there is a better way...
When you call `UIDocument.ShowElements`, the active `Document` will change to the document you are showing elements in, therefore:
If you filter for any element in one of the views of the newly created document, you can call it using that element to activate that document.
Sometimes you get the dialogue 'No good view found', which is odd, considering you know there is at least one view with the element in considering you are filtering for it by view. You can handle the appearance of this dialogue and the `ActiveDocument` still gets changed. I believe the new document generally has a view with `Elevation` markers, hence there is always something to find and show.
I don't know the implications of changing the `ActiveDocument` this way or why the API has no ability to directly change the `ActiveDocument` directly, but I suspect if it were easy in terms of how the API works, it would have been done by now.
This workaround was also mentioned in The Building Coder discussion
on [mirroring in a new family and changing active view](http://thebuildingcoder.typepad.com/blog/2010/11/mirroring-in-a-new-family-and-changing-active-view.html):
> The first issue that arises is that the mirror command requires a current active view, which is not automatically present in the family document. Joe discovers a workaround for that issue using the `ShowElements` method. It generates an unwanted warning message, so a second step is required to deal with eliminating that as well.
> As you can see on reading the final solution carefully, you can use the `ShowElements` method to change the active view and even switch it between the family and project documents. The official Revit 2011 API does not provide any method to switch the active view, but using `ShowElements` can be used to create a workaround for that."
I implemented a new external
command [CmdSwitchDoc](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdSwitchDoc.cs)
in [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples) to try this out, in
[release 2018.0.138.1](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.138.1).
It demonstrates two uses of the `ShowElements` method:
- [Zoom to selected elements](#3)
- [Toggle between documents and views](#4)
#### Zoom to Selected Elements
```csharp
///
/// Zoom to the given elements, switching view if needed.
/// summary>
/// param>
/// Error message on failureparam>
/// Elements causing failureparam>
/// returns>
Result ZoomToElements(
UIDocument uidoc,
ICollection ids,
ref string message,
ElementSet elements )
{
int n = ids.Count;
if( 0 == n )
{
message = "Please select at least one element to zoom to.";
return Result.Cancelled;
}
try
{
uidoc.ShowElements( ids );
}
catch
{
Document doc = uidoc.Document;
foreach( ElementId id in ids )
{
Element e = doc.GetElement( id );
elements.Insert( e );
}
message = string.Format(
"Cannot zoom to element{0}.",
1 == n ? "" : "s" );
return Result.Failed;
}
return Result.Succeeded;
}
```
This functionality is similar to that provided by
the [Zoom to Awesome! add-in](https://bimopedia.com/2013/04/02/zoom-to-awesome) by Phil Read.
Here is a 40-second demo by Luke Johnson
of [using Zoom to Awesome](https://knowledge.autodesk.com/support/revit-products/getting-started/caas/screencast/Main/Details/8e9a043d-9383-496b-8e86-6ec3ab055c0e.html),
also showing how to add a keyboard shortcut:

#### Toggle Between Documents and Views
```csharp
///
/// Toggle back and forth between two different documents
/// summary>
void ToggleViews(
View view1,
string filepath2 )
{
Document doc = view1.Document;
UIDocument uidoc = new UIDocument( doc );
Application app = doc.Application;
UIApplication uiapp = new UIApplication( app );
// Select some elements in the first document
ICollection idsView1
= new FilteredElementCollector( doc, view1.Id )
.WhereElementIsNotElementType()
.ToElementIds();
// Open the second file
UIDocument uidoc2 = uiapp
.OpenAndActivateDocument( filepath2 );
Document doc2 = uidoc2.Document;
// Do something in second file
using( Transaction tx = new Transaction( doc2 ) )
{
tx.Start( "Change Scale" );
doc2.ActiveView.get_Parameter(
BuiltInParameter.VIEW_SCALE_PULLDOWN_METRIC )
.Set( 20 );
tx.Commit();
}
// Save modified second file
SaveAsOptions opt = new SaveAsOptions
{
OverwriteExistingFile = true
};
doc2.SaveAs( filepath2, opt );
// Switch back to original file;
// in a new file, doc.PathName is empty
if( !string.IsNullOrEmpty( doc.PathName ) )
{
uiapp.OpenAndActivateDocument(
doc.PathName );
doc2.Close( false ); // no problem here, says Remy
}
else
{
// Avoid using OpenAndActivateDocument
uidoc.ShowElements( idsView1 );
uidoc.RefreshActiveView();
//doc2.Close( false ); // Remy says: Revit throws the exception and doesn't close the file
}
}
```
![Toggle](img/toggle.png)