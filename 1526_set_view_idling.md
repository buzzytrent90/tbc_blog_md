---
post_number: "1526"
title: "Set View Idling"
slug: "set_view_idling"
author: "Jeremy Tammik"
tags: ['elements', 'filtering', 'references', 'revit-api', 'sheets', 'transactions', 'views', 'windows']
source_file: "1526_set_view_idling.md"
original_url: "https://thebuildingcoder.typepad.com/blog/1526_set_view_idling.html"
---

### Setting Active View During Idling
Here is a summary of the discussion and solution
for [setting `ActiveView` during the Idling event](http://forums.autodesk.com/t5/revit-api-forum/setting-activeview-during-idling-event/m-p/6848256) from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) raised
and solved by Rudi 'Revitalizer' and Kinjal Desai, founder and fullstack developer
at [Dwaravati](http://www.dwaravati.com):
\*\*Question:\*\* I need to set `ActiveView` in `Idling` event handler.
As per API documentation, this operation should not be invalid: no open transactions; `IsModifiable` is ok; `IsReadOnly` is ok; No pre-action events around.
However, trying to do so throws an `InvalidOperationException` with the message "Setting active view is temporarily disabled".
Closest explanation I could reach to is
Jeremy's [casual reply](http://thebuildingcoder.typepad.com/blog/2011/09/activate-a-3d-view.html#comment-2097342515) "the system may be busy with something else"
to a [blog comment](http://thebuildingcoder.typepad.com/blog/2011/09/activate-a-3d-view.html#comment-2097342519)
on [activating a 3D View](http://thebuildingcoder.typepad.com/blog/2011/09/activate-a-3d-view.html).
In a sense, that is the case – the system is 'busy', because the `Idling` event is being processed at the moment.
However, I wonder if that is the real reason or a real limitation in this case – for the system is also busy when `IExternalCommand.Execute` is being processed, in which case setting `ActiveView` functions perfectly.
Looking forward to expert comments on whether setting `ActiveView` is indeed invalid in `Idling` under all circumstances and whether there is a way around it.
\*\*Answer 1:\*\* As a workaround, would it be possible to select the view (using the API), then use `PostCommand` to set the active view? Then wait for idling again...
\*\*Answer 2:\*\* There is a `UIDocument.RequestViewChange` method made for the case you described.
It is included in the [Revit 2015 API](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html)
[Document API additions](http://thebuildingcoder.typepad.com/blog/2014/04/whats-new-in-the-revit-2015-api.html#4.02):
#### UIDocument Operations and Additions

The new method UIDocument.RequestViewChange requests to change the active view by posting a message asynchronously. Unlike setting the ActiveView property, this will not make the change in active view immediately. Instead, the request will be posted to occur when control returns to Revit from the API context. This method is permitted to change the active view from the Idling event or an ExternalEvent callback.

\*\*Response:\*\* Thank you for all your suggestions and for bringing this to light.
It is async, but that's clearly the author's intended way; I'll fit my logic around it.
Imho, the API reference would be more useful (and save some hours and brain tissues) if such 'sister' functions could be linked in 'See Also' or something.
I implemented the suggestion using `RequestViewChange`.
Using a async call wasn't ideal for my specific need, but I refactored my code to accommodate it.
In absence of any answers, my fall-back alternative (far from ideal, but functional nonetheless), would have been to fire `Ctrl+F4` keystrokes (conditionally) from the `Idling` event. I am not fan of this approach, but it would also have served for my specific use case.
`RequestViewChange` is definitively the API authors' intended way to achieve this.
I attached a sample model containing macros to demonstrate both the problem and the solution implementing the permissible and intended way to change active view from `Idling`.
It is a project file with a blank model with two floor plans, containing the code (fairly simple and self-explanatory) in document macros.
On launching the `Demo_SetActiveViewInIdling` macro, a modeless form is displayed:
![Demo_SetActiveViewInIdling main form](img/set_view_idling_1.png)
If you choose the permissible working solution, a message is displayed announcing what will happen, and the action is successfully performed:
![Demo_SetActiveViewInIdling successful operation coming up](img/set_view_idling_2.png)
If you attempt the problematic approach, this is also announced in advance:
![Demo_SetActiveViewInIdling problematic approach coming up](img/set_view_idling_3.png)
The problematic approach throws an exception, which is handled and reported as well:
![Demo_SetActiveViewInIdling error report](img/set_view_idling_4.png)
The macro mainline just launches the modeless form that does all the work:
```csharp
public void Demo_SetActiveViewInIdling()
{
FloatingForm form = new FloatingForm();
form.Setup( Application );
form.Show();
}
```
Here is the actual working code implemented by `FloatingForm`:
```csharp
namespace SetActiveViewInIdling
{
///
/// Description of FloatingForm.
/// summary>
public partial class FloatingForm : WinForm.Form
{
bool attemptSettingActiveView_forbidden;
bool attemptSettingActiveView_async;
bool deregister;
#region Operational & misc
public FloatingForm()
{
//
// The InitializeComponent() call is required for Windows Forms designer support.
//
InitializeComponent();
this.FormClosing += FloatingForm_FormClosing;
}
private void FloatingForm_FormClosing( object sender, WinForm.FormClosingEventArgs e )
{
deregister = true;
}
UIApplication uiApplication;
internal void Setup( UIApplication uiApplication )
{
this.uiApplication = uiApplication;
this.uiApplication.Idling += Application_Idling;
}
#endregion
private void demoButton_Click( object sender, EventArgs e )
{
//this flag will be picked by next call to our Idling event handler
attemptSettingActiveView_forbidden = true;
}
private void demoButton2_Click( object sender, EventArgs e )
{
//this flag will be picked by next call to our Idling event handler
attemptSettingActiveView_async = true;
}
private void Application_Idling( object sender, Autodesk.Revit.UI.Events.IdlingEventArgs e )
{
if( attemptSettingActiveView_forbidden )
{
try
{
attemptSettingActiveView_async = false;
attemptSettingActiveView_forbidden = false;
var arbitaryFloorPlan = new FilteredElementCollector( uiApplication.ActiveUIDocument.Document ).OfClass( typeof( ViewPlan ) ).Cast().Where( x => !x.IsTemplate ).FirstOrDefault();
if( arbitaryFloorPlan != null )
{
TaskDialog.Show( "SetActiveViewInIdling sample", "Message from within Application_Idling()\n\nNext statement is attempting to set \'" + arbitaryFloorPlan.ViewName + "\' as UIDocument.ActiveView" );
uiApplication.ActiveUIDocument.ActiveView = arbitaryFloorPlan;
}
else
TaskDialog.Show( "SetActiveViewInIdling sample", "Add one or more Floor Plans." );
}
catch( Autodesk.Revit.Exceptions.InvalidOperationException invalidOperationException )
{
TaskDialog.Show( "SetActiveViewInIdling sample", "\*\*\*Issue Reproduced\*\*\*\n\nMessage: " + invalidOperationException.Message + "\n\nFor more details, attach debugger, breakopint at FloatingForm.Application_Idling > catch." );
}
}
else if( attemptSettingActiveView_async )
{
attemptSettingActiveView_async = false;
attemptSettingActiveView_forbidden = false;
var arbitaryFloorPlan = new FilteredElementCollector( uiApplication.ActiveUIDocument.Document ).OfClass( typeof( ViewPlan ) ).Cast().Where( x => !x.IsTemplate ).FirstOrDefault();
TaskDialog.Show( "SetActiveViewInIdling sample", "Message from within Application_Idling()\n\nNext statement will request \'" + arbitaryFloorPlan.ViewName + "\' to be activated (using UIDocument.RequestViewChange()) asynchronously." );
uiApplication.ActiveUIDocument.RequestViewChange( arbitaryFloorPlan );
}
else if( deregister )
{
this.uiApplication.Idling -= Application_Idling;
}
}
}
}
```
Many thanks to Kinjal for implementing and sharing this nice demo solution, and to Matt Taylor and above all Rudi 'Revitalizer' for their good suggestions!